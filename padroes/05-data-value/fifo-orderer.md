# Padrão: Ordenador FIFO (FIFO Orderer)

## 1. Resumo (O que é?)
O padrão de Ordenador FIFO foca primariamente em transmitir e inserir ordens (Primeiro a Entrar, Primeiro a Sair - *First in, First out*) num formato pragmático para ecossistemas e negócios que não necessitam de volumes massivos de escalabilidade diária. Ele age item por item.

## 2. O Problema
* Em uma transmissão de streaming (ou requisições de serviço), a lógica para o ecossistema externo de destinos só suporta e respeita os dados se eles chegarem em absoluta cronologia real. 
* Não é justificável desenhar *Bin Pack Orderers* (empacotamento de maletas e agrupamento prévio) ou introduzir gargalos massivos via rede, porque a aplicação exige baixíssima latência unitária e os logs não geram trilhões de dados por segundo no cluster. 

## 3. A Solução
Simplifique, relaxando as regras de otimização da rede do pipeline de lote: não gerencie ou crie lotes volumosos de transação (Bulk APIs). Em vez disso, detecte a gravação de entrada, construa a requisição e faça um Push de transmissão único (tamanho do Lote = 1 ou `max.in.flight.requests = 1`). A sua configuração garante ao sistema (seja num Kafka Producer ou similar) aguardar o sinal explícito de reconhecimento (ACK - Acknowledgment) do item antes de enviar o próximo da fila. O próximo só entrará no canal se o item original transacionar no broker; se não, sofrerá retentativas infinitas de tráfego que, ironicamente, trancam fisicamente a passagem sem exigir codificação complexa.

## 4. Consequências e Trade-offs
* **Vantagens:** Abordagem universal para regras restritas. Extremamente fácil de habilitar via arquivos de configuração simples em provedores (Kafka/AWS).
* **Desvantagens/Atenção:** 
  * **Custo Exorbitante do I/O na Rede:** Se houver repentinamente dezenas de milhões de itens, enviar um log, aguardar confirmação via rede por IP transatlântica (Latência) e em seguida abrir novo canal TCP para repassar, trancará o processo analítico irremediavelmente (sufoco de conexão de infraestrutura).
  * **Não Significa Entrega Unicamente (Exactly-Once):** FIFO é cronologia, não volume fixo. Um arquivo foi para a rede com sucesso, mas o sinal "ACK" final da API caiu. O sistema não o ouviu, aciona as regras de Retentativa e grava exatamente a mesma cópia do erro em ordem com segurança na base (duplicação). Requer controle à parte.

## 5. Exemplo de Aplicação Prática
Um robô notificador envia SMS ao gerente por movimentações ilegais dentro das transações contábeis. Como existem não mais do que 1 ou 2 alertas e é gravíssimo a ordem das movimentações estarem cruzadas na notificação de fraude, ele emite alertas simples ao Gateway através do FIFO via conexões isoladas por minuto. 

## 6. Exemplo Simples de Código
```python
# Sem paralelismo ou malotes Bulk: Arquitetura Kafka simples garantida por SDK
produtor = Producer({
    'max.in.flight.requests.per.connection': 1, # Processa o voo 1 por 1
    'enable.idempotence': True, # Segurança opcional da AWS
})

for registro in relatorio_fifo:
    produtor.produce(registro)
    produtor.flush() # Congela a execução explícita (bloqueio do fluxo) aguardando SUCESSO
```

## 7. Padrões Relacionados ou Nomes Similares
É um oposto extremo do ganho elástico em nuvem demonstrado pelo *Bin Pack Orderer*. Dependente dos mecanismos do *Idempotency Patterns* (Padrões de Idempotência) para cobrir falhas de segurança do I/O de rede.
