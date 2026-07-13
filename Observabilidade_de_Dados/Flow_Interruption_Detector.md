# Flow Interruption Detector

**Categoria:** Observabilidade de Dados

## 🎯 Objetivo
Alertar com eficácia extrema na orquestração pura as paralisações ativas orgânicas silenciosas ("Silent failures") na fonte originária que abortam a propagação do fluxo, inviabilizando na raiz da base as atuações e os painéis de consumo sem interrupções abruptas lógicas no framework do motor.

## O Problema
Tens as tuas maravilhosas regras de `Audit` ativas (AWAP), e contudo ontem os clientes reclamaram. Motivo: Nenhuns dados chegaram ativamente às 8h da manhã. Foste verificar e não houve falhas de job ou timeouts. Como? Porque o stream simplesmente deixou estaticamente de receber dados e gravou "0 bytes" no S3 sem despoletar anomalias ativas (um silent error de infraestrutura no produtor originário orquestrador não visível nas tuas pipelines de processamento passivo abstrato).

## A Solução
Se o fluxo quebra por inatividade estrita adotas na observabilidade pura os moldes do "Flow Interruption Detector". As tuas pipelines contínuas orgânicas ativam e geram regras de disparo abstrato por janelas purificadoras ou avaliam latências em janelas vazias de envio contínuo orgânico. Na vertente em Cloud nativa passiva analítica as ferramentas leem instintivamente ou interrogam no DW orquestrado a metadata pura (Last Modified date) das tabelas (`pg_xact_commit_timestamp` nas views estritas ou `last_version` no History do Delta) e submetem a métrica pura analítica de "Frescura" ao Grafana/Prometheus estrito orgânico que acciona os sinos de "Missing data flow".

## Prós e Contras
- **Prós:** 
  - Restabelece de imediato a monitorização crítica na base originária que falhava em falhas passivas silenciosas, dando confiança pura estrita orquestrada perante a garantia nativa orgânica nas bases analíticas dos consumidores.
  - Ferramentas analíticas e tabelas cloud geridas contêm frequentemente de graça ativada no background a vertente purificada das informações lógicas (`INFORMATION_SCHEMA.PARTITIONS`), não carecendo cálculos intrusivos estritos na arquitetura de data ingestion original.
- **Contras:** 
  - Propicia horrivelmente em fluxos irregulares (e.g. vendas com pausas noturnas puras ativas) os engarrafamentos estritos puros orgânicos na orquestração dos falsos positivos no painel levando nativamente ativados aos ignoros drásticos dos operadores cansados em cloud (Alarm Fatigue puros originais estritos).
  - É refém orgânico subjacente nativo das anomalias do Housekeeping (Compaction process puro) visto que limpezas nativas geram ativamente ecos na Metadata de Timestamp alterando a data da base mas os dados subjacentes purificados e novos na realidade nativa orquestradora são estritamente nulos.