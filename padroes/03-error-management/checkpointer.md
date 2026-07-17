# Padrão: Ponto de Controle (Checkpointer)

## 1. Resumo (O que é?)
O padrão de Ponto de Controle (Checkpointer) é uma ferramenta de tolerância a falhas primariamente associada a pipelines de fluxo contínuo (streaming). Ele se encarrega de salvar regularmente o estado do processamento e a última posição lida na origem em um armazenamento persistente, garantindo que o sistema saiba de onde recomeçar se a aplicação cair.

## 2. O Problema
* Em um fluxo contínuo (streaming), você recebe e processa dados initerruptamente (sem o conceito finito de um arquivo ou lote diário).
* Se o servidor que roda sua aplicação travar de forma inesperada (falha fatal), a aplicação reiniciará, mas não saberá onde parou. Sem um controle de progresso, ela voltaria a reprocessar os dados do início, causando desordem e dados massivamente duplicados.

## 3. A Solução
Sua aplicação deve registrar periodicamente o deslocamento (*offset* ou marca de progresso) e qualquer estado em memória (como contagens parciais) em um sistema externo resiliente. Ferramentas como Apache Spark Structured Streaming ou Apache Flink automatizam a criação desses checkpoints com arquivos em Data Lakes (S3/GCS) ou sistemas dedicados, salvando pequenos resumos com o nome/número da iteração processada. Ao reiniciar, o sistema lê o último checkpoint com sucesso e retoma a leitura de onde havia parado.

## 4. Consequências e Trade-offs
* **Vantagens:** Recuperação a partir de falhas quase transparente; essencial para prover garantias de entrega como "pelo menos uma vez" ou o almejado "exatamente uma vez" (com a ajuda de bancos idempotentes).
* **Desvantagens/Atenção:** 
  * **Latência vs Frequência:** Checkpoints adicionam lentidão. Fazer checkpoint a cada milissegundo garante perda quase nula, mas freia o fluxo. Fazer a cada 1 hora otimiza o fluxo, mas obriga o sistema a reprocessar até 1 hora de dados caso falhe.
  * **Sensação Falsa de "Exatamente Uma Vez":** Sozinho, o Checkpointer não garante a entrega perfeitamente única. Se um lote processar e gravar dados com sucesso, mas a aplicação morrer um milissegundo antes de gravar a *confirmação* no checkpoint, ao religar o sistema, ele reprocessará aquele lote e regravará os mesmos dados no destino.

## 5. Exemplo de Aplicação Prática
Um painel ao vivo conta os usuários no site a cada 10 minutos. O sistema sofre uma pane de hardware. Em vez de recomeçar a contagem do zero desde de manhã e estragar o gráfico, a aplicação lê o checkpoint do minuto anterior no S3, resgata as contagens prévias, descobre qual linha do log de origem ainda não leu, e segue em frente sem que os usuários do painel percebam a queda brusca.

## 6. Exemplo Simples de Código
```python
# Habilitando Checkpointing no Spark Structured Streaming
fluxo_saida = (
    fluxo_processado.writeStream
    .outputMode('update')
    # O diretório 'checkpoint' guardará com segurança a linha do tempo do fluxo
    .option('checkpointLocation', 's3://meu-bucket/app-checkpoints')
    .start()
)
```

## 7. Padrões Relacionados ou Nomes Similares
Necessita frequentemente trabalhar em conjunto com padrões de idempotência como o *Transactional Writer*, *Keyed Idempotency* ou *Windowed Deduplicator* para prevenir as duplicações de *at-least-once processing* derivadas das falhas no commit dos checkpoints.
