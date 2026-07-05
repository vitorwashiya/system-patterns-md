
# Change Data Capture

## Classificação
- **Tipo:** Comportamental
- **Escopo:** Objeto

## Intenção
Acompanhar ativamente e transmitir fluxos ininterruptos de alterações atômicas derivadas estritamente do log primário de um banco de dados, assegurando assim mínima latência em propagação física para ecossistemas consumidores remotos.

## Também Conhecido Como
CDC.

## Motivação (Contexto)
Consumidores exigem capacidades exploratórias sub-minuto (quase tempo real) em sistemas transacionais legados supercríticos, aposentando carregamentos diários exaustivos em favor da emissão reativa granular de operações diretas de manipulação física (inserts, updates agressivos, hard deletes).

## Aplicabilidade
- Use o Change Data Capture quando:
  - Latências permitidas de processamento são rigidamente aferidas na casa dos sub-minutos.
  - A interceptação rigorosa de deleções diretas (hard deletes) e imagens pré/pós atualização se mostram vitais e insubstituíveis.
  - O motor do banco relacional de origem dispor e autorizar leituras ininterruptas do mecanismo basilar nativo de commit logs.

## Estrutura e Participantes
- **Commit Log / Transaction Log:** O coração nativo do provedor, contendo o traço operacional cru de todos os registros fáticos processados localmente.
- **CDC Connector:** Componente auxiliar remoto de monitoramento e parser de log contínuo isolando o tráfego lógico (ex: Kafka Connect Debezium).
- **Streaming Broker:** Hub persistente receptivo operando a disseminação de semântica streaming (Kafka).

## Colaborações
O Storage Relacional persiste silenciosamente operações no Commit Log. Em paralelo assíncrono rigoroso, o CDC Connector interroga ciclicamente as margens deste log e emite instantaneamente eventos atômicos envelopados contendo o registro transacional físico até o Streaming Broker.

## Consequências
- **Prós:**
  - Redução assombrosa das janelas de defasagem (latência garantida ínfima na chegada).
  - Mitiga eficientemente hiatos complexos resolvendo a captura exata autêntica de hard deletes transacionais e edições.
- **Contras:**
  - Amplifica dramaticamente os obstáculos arquiteturais em estabilidade do Connector na operação isolada in-house.
  - Complexifica pesadamente as resoluções de joins de leitura: injeta de supetão semântica mutante (data-in-motion) dificultando inferências SQL relacionais antigas (data-at-rest).

## Implementação (Exemplo de Código)
```json
// O paradigma puramente declarativo exigido em Conectores CDC Kafka (Debezium / PostgreSQL)
{
  "name": "visits-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres-master-internal",
    "database.port": "5432",
    "database.user": "cdc_replica_user",
    "database.dbname": "production_visits",
    "database.server.name": "dbserver1_prod",
    "schema.include.list": "dedp_schema",
    "topic.prefix": "cdc_outbox"
  }
}
```

## Padrões Relacionados

* **Incremental Loader:** Principal contraponto rebaixado ao qual se acorre quando exigências duras temporais são suspensas permitindo lotes maciços ao invés de fluxos diretos.

---
