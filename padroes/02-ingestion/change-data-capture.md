# Padrão: Captura de Dados Alterados (Change Data Capture - CDC)

## 1. Resumo (O que é?)
A Captura de Dados Alterados (CDC) é um padrão que escuta diretamente o log de transações de um banco de dados para capturar e transmitir todas as modificações de dados em tempo real.

Em vez de executar consultas agendadas para descobrir o que mudou, o CDC age nos bastidores, transmitindo eventos de inserção, atualização e exclusão à medida que acontecem, proporcionando uma latência muito menor na ingestão.

## 2. O Problema
* Os consumidores de dados reclamam da demora para visualizar novas informações e exigem latência próxima do tempo real (ex: dados disponíveis em até 30 segundos).
* O padrão de Carga Incremental tradicional é muito lento ou não captura exclusões físicas (hard deletes) na tabela de origem.

## 3. A Solução
O padrão CDC resolve o problema conectando-se ao log de commit (commit log) do banco de dados de origem. Este log grava sequencialmente todas as operações de banco de dados. Um conector lê este log continuamente e publica as alterações em um serviço de mensageria (como Apache Kafka). A partir dali, os consumidores podem atualizar suas próprias bases ou realizar análises mantendo um reflexo exato do banco original.

## 4. Consequências e Trade-offs
* **Vantagens:** Baixíssima latência na ingestão, captura exata de inserções, atualizações e todas as exclusões (mesmo físicas), sem sobrecarregar o banco de dados com consultas pesadas.
* **Desvantagens/Atenção:** 
  * **Complexidade de Setup:** Requer privilégios administrativos no banco de dados e configuração de ferramentas externas (como Debezium), exigindo frequentemente ajuda da equipe de infraestrutura.
  * **Escopo de Dados:** Pode ser difícil resgatar o histórico completo de alterações passadas, focando apenas nos eventos ocorridos após a configuração do conector.

## 5. Exemplo de Aplicação Prática
Um sistema de comércio eletrônico usa um banco de dados relacional para gerenciar o estoque. Um conector CDC escuta as mudanças nessa base de dados e envia notificações instantâneas para um sistema de recomendações sempre que um produto é vendido ou tem seu preço alterado, permitindo que a vitrine seja ajustada imediatamente.

## 6. Exemplo Simples de Código
```json
// Exemplo de configuração de um conector CDC (Debezium)
{
  "name": "vendas-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "banco_origem",
    "database.user": "usuario_cdc",
    "database.dbname": "ecommerce",
    "table.include.list": "public.estoque",
    "topic.prefix": "cdc_eventos"
  }
}
```

## 7. Padrões Relacionados ou Nomes Similares
Conhecido amplamente pela sigla *CDC*. Ferramentas populares incluem *Debezium*, *AWS DMS* e os recursos de *Change Data Feed (CDF)* em formatos de tabela como Delta Lake. Ele substitui ou evolui a abordagem de *Incremental Loader*.
