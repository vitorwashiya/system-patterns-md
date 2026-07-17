# Padrão: Desduplicador em Janela (Windowed Deduplicator)

## 1. Resumo (O que é?)
O Desduplicador em Janela é uma técnica utilizada para garantir o processamento exato de registros em cenários onde mensagens duplicadas podem ocorrer. Para limitar o consumo de recursos na identificação de duplicatas, este padrão aplica a desduplicação observando uma janela temporal limitada de registros já processados.

## 2. O Problema
* Seus sistemas originais ou conexões de rede frequentemente falham e geram repetições da mesma mensagem (garantia "at-least-once" ou "pelo menos uma vez").
* Inserir os mesmos dados múltiplas vezes corrompe os resultados e confunde os consumidores finais. Contudo, procurar por duplicatas na história inteira de um sistema contínuo (streaming) exigiria memória infinita.

## 3. A Solução
O padrão introduz uma lógica de filtragem que descarta registros com identificadores (chaves) que já foram vistos recentemente. No processamento em lote (batch), a "janela" geralmente é o próprio conjunto de dados que está sendo processado na execução atual. No processamento em fluxo contínuo (streaming), o sistema mantém as chaves processadas em um armazenamento de estado (state store) temporário. O limite desse armazenamento é definido por um tempo (janela). Se uma mensagem idêntica chegar dentro dessa janela, ela é descartada; chaves mais velhas que a janela são apagadas para liberar memória.

## 4. Consequências e Trade-offs
* **Vantagens:** Protege a integridade dos dados finais garantindo o processamento único, mantendo o consumo de memória sob controle.
* **Desvantagens/Atenção:** 
  * **Espaço vs Tempo:** Escolher o tamanho da janela é um desafio. Uma janela pequena gastará pouca memória, mas poderá deixar passar uma mensagem duplicada que chegar com muito atraso. Uma janela enorme pegará os atrasados, mas demandará forte capacidade de hardware (RAM/Disco).
  * **Não garante entrega "Exatamente Uma Vez":** Se o próprio job de desduplicação falhar antes de registrar o resultado no destino, ao reiniciar, ele pode reprocessar e acabar inserindo o dado novamente. Muitas vezes precisa ser combinado com gravações idempotentes.

## 5. Exemplo de Aplicação Prática
Um serviço de streaming de vídeo gera "eventos de play". Devido a falhas no 4G dos usuários, o aplicativo de celular manda três vezes o mesmo evento com o mesmo `evento_id`. O job no servidor mantém uma memória dos `evento_id` dos últimos 10 minutos. Ao ver os eventos duplicados chegarem logo em seguida, ele os ignora e salva apenas a primeira visualização.

## 6. Exemplo Simples de Código
```python
# Desduplicação em janela usando Apache Spark Structured Streaming
eventos_limpos = (
    fluxo_de_dados
    # Lembra os eventos por 10 minutos (a Janela/Watermark)
    .withWatermark('horario_evento', '10 minutes')
    # Descarta se o mesmo ID for visto novamente nesse intervalo
    .dropDuplicates(["evento_id", "horario_evento"])
)
```

## 7. Padrões Relacionados ou Nomes Similares
Muitas vezes está por trás da funcionalidade *Exactly-Once Processing* (processamento exatamente uma vez). Beneficia-se de *Keyed Idempotency* (Idempotência Baseada em Chave) para proteção fim a fim.
