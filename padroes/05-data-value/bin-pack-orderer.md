# Padrão: Organizador de Caixas (Bin Pack Orderer)

## 1. Resumo (O que é?)
O padrão Organizador de Caixas é focado em otimização do tráfego de rede e garantias difíceis de ordenação de dados, especialmente para sistemas sensíveis à desordem de chegada. Ele atua agrupando pacotes (records) logicamente em caixas menores antes de inserções pesadas no destino.

## 2. O Problema
* O pipeline escreve lotes para os fornecedores externos que expõem APIs tolerantes, que podem emitir retentativas e sofrer "Commits Parciais" (ex: um envio com 3 registros tem os registros 1 e 3 gravados no banco de dados com sucesso e o 2 falha por desconexão momentânea).
* Como o sistema transacional deve respeitar cronologias sensíveis (ex: um rastreamento de veículos nas vias ou sistemas fiscais de conta bancária), a nova tentativa atrasada do erro (tentar subir o erro número 2 horas depois do 3 estar na base) pode quebrar as relações lógicas da aplicação a jusante.

## 3. A Solução
O padrão previne inconsistência baseada em Partial Commits da seguinte maneira: 
1. **Otimização:** A origem executa uma operação interna para reunir todos os registros e classificá-los, mantendo as chronologias e criando *Bins* (gavetas) perfeitamente separadas para chaves (IDs da regra do negócio) exatas; garantindo uma única entidade para cada agrupamento em memória.
2. **Emissão:** Esse lote de tamanho ideal, empacotado individualmente, é então enviado por lote em massa para a API externa e transacionado com persistência paralela isolada.
3. **Imunidade Sistêmica:** Por design, não existem outros registros ou tentativas relacionadas da entidade dentro da transação além dos itens em sua própria maleta. Como a regra restringe o paralelismo para as caixas do mesmo usuário, atrasos eventuais numa inserção só afetarão o tempo geral, pois o próximo "Bin" contendo a evolução cronológica estará bloqueado na fila do produtor aguardando que essa transação que teve retry termine antes.

## 4. Consequências e Trade-offs
* **Vantagens:** Otimização altíssima ao realizar chamadas em larga escala (bulk / lote), suportando lógicas severas em APIS fracamente consistentes.
* **Desvantagens/Atenção:** 
  * **Retentativas Sistêmicas (Retries) a Longo Prazo:** Apesar da maleta garantir o sucesso linear no fluxo, uma falha de proporções colossais derrubando todo o pipeline envolveria retransmissões repetidas das mesmas maletas pela rede.
  * **Complexidade Pessoal:** Construir as logísticas exatas de organização é complexo para o desenvolvedor porque vai além da chamada explícita de `ORDER BY`, lidando muitas vezes com agrupamentos granulares da linguagem de programação manualmente para a arquitetura de rede.

## 5. Exemplo de Aplicação Prática
Se uma viagem de táxi gera posições GPRS de minuto em minuto, a API terceirizada recebe esses "pacotes". Se falharem durante uma requisição HTTP da frota mista, um *Bin Pack Orderer* fará com que uma "bolsa" em sua memória aloque todos os pontos em lista estrita da viagem A1 e bloqueie novas tentativas relacionadas em vez de jogar tudo em lote no destino da nuvem. Sincronismo garantido.

## 6. Exemplo Simples de Código
```python
# Lógica Python local isolando cada maleta (Bin) e os enfileirando (Ordem)
grupos_entrega = []
id_visitante_atual = None
for registro_visit in registros_ordenados_historicamente:
    if registro_visit.id != id_visitante_atual:
        id_visitante_atual = registro_visit.id
        grupos_entrega.append([]) # Nova mala
    
    # Coloca no final do bin pack atual
    grupos_entrega[-1].append(registro_visit)

# Executa sequencialmente o disparo de todos os pacotes garantindo fluxo correto
enviar_apis(grupos_entrega)
```

## 7. Padrões Relacionados ou Nomes Similares
Uma alternativa muito mais barata (em termos de chamadas via HTTP API/Network) ao *FIFO Orderer*.
