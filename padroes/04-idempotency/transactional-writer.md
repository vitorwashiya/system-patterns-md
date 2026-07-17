# Padrão: Gravador Transacional (Transactional Writer)

## 1. Resumo (O que é?)
O Gravador Transacional confia nos recursos transacionais do banco de dados (propriedades ACID) para implementar a garantia de que as alterações feitas por um produtor sejam expostas aos consumidores na íntegra ou não sejam expostas (Tudo ou Nada). O *commit* da transação atua como uma barreira que impede que leitores vejam dados processados pela metade.

## 2. O Problema
* O seu pipeline se aproveita de instâncias de nuvem com capacidade ociosa (Spot Instances) que são mais baratas, mas podem ser desligadas a qualquer momento.
* Sempre que o provedor desliga a máquina, as tarefas em execução caem e reiniciam. O resultado são consumidores reclamando de registros duplicados e incompletos chegando na base.

## 3. A Solução
Utilizar a capacidade transacional nativa da camada de armazenamento para que os dados "em progresso" não sejam visíveis aos consumidores até a tarefa inteira acabar. Em bancos relacionais ou formatos modernos (Delta, Iceberg), isso envolve iniciar uma transação (`BEGIN`), gravar os dados e finalizar com uma instrução de confirmação (`COMMIT`). Se o processamento for derrubado antes do commit, ocorre o *rollback* e o banco de dados simplesmente ignora ou descarta os blocos parcialmente processados.

## 4. Consequências e Trade-offs
* **Vantagens:** Protege os consumidores de leituras parciais ou sujas, transferindo a complexidade do gerenciamento de estado parcial para a tecnologia do banco.
* **Desvantagens/Atenção:** 
  * **Latência Adicional:** Consumidores não conseguem ver arquivos brutos ou registros parciais na hora em que caem no disco; precisam esperar que a transação inteira do bloco feche, adicionando latência ponta a ponta.
  * **Retentativas e Idempotência:** Sozinha, a transação não garante idempotência. Se a tarefa sofrer falha após o commit, e rodar novamente com sucesso, a transação criará duas versões iguais. Requer padrões auxiliares para evitar a inserção repetida na re-execução.
  * **Suporte da Ferramenta:** Muitos sistemas distribuídos (ex: Spark sem Delta) não suportam transações nativas unificadas, dificultando o uso puro do padrão.

## 5. Exemplo de Aplicação Prática
Um job diário calcula bônus financeiros. Ele usa *Transactional Writer* abrindo uma transação, processando milhares de clientes e fazendo o commit somente no fim. Se a máquina falhar na metade, nenhum bônus será exibido para ninguém, evitando que a equipe de atendimento lide com "Alguns clientes receberam o bônus, outros não".

## 6. Exemplo Simples de Código
```python
# Habilitando produtor transacional no Apache Kafka (via Flink, por exemplo)
kafka_sink = (
    KafkaSink.builder()
    .set_bootstrap_servers("localhost:9094")
    # Configura a entrega baseada em Transações: Exatamente Uma Vez
    .set_delivery_guarantee(DeliveryGuarantee.EXACTLY_ONCE)
    .set_property('transaction.timeout.ms', '60000') # 1 minuto
    .build()
)
```

## 7. Padrões Relacionados ou Nomes Similares
Essencial para implementar processamento do tipo *Exactly-Once* (exatamente uma vez). Frequentemente atua juntamente com *Keyed Idempotency* e padrões de reescrita como *Data Overwrite*.
