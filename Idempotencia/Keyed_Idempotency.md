# Keyed Idempotency

**Categoria:** Idempotência

## 🎯 Objetivo
Gerar processamentos inabaláveis a nível de integridade ao assegurar entregas puras e perfeitamente deduplicadas nas bases de apoio sem interversão contínua das plataformas subjacentes por retries, gerando chaves identificadoras exclusivas por instâncias.

## O Problema
O teu motor baseia-se num recetor transacional. Lidas diariamente com "exactly-once" prometidos que perante retries acabam a enviar os mesmos resultados a um registo ou recetor (Kafka Sink). Queres atenuar essa duplicação intrusiva causada diretamente nas rotinas da lógica de gravação após reinícios provocados no motor ou no provedor.

## A Solução
Focando-se não no fluxo mas nos sistemas transacionais e de base Key-Value (ex: ScyllaDB, HBase, Redis, Dynamo), implementas a Keyed Idempotency estática baseando as escritas através da conceção e criação estrita de uma chave artificial totalmente imutável, como a "hora de entrada anexada (append time)" originada pelas meta-informações do produtor inicial em contraponto à utilização da perigosa janela transitória "hora de processamento". Assim, se o job re-tenta, reprocessa as tabelas substituindo os registos nos IDs iguais, agindo puramente na substituição direta transparente.

## Prós e Contras
- **Prós:** 
  - Estratégia sem impacto nos consumos laterais ou necessitando read-and-writes excessivas para a garantia de não repetição.
  - Oferece deduplicação intrínseca perante qualquer número exaustivo indesejado de falhas das threads do Apache Spark ou do Flink ao repousar no UPSERT silencioso baseado em chaves determinísticas de base.
- **Contras:** 
  - Restringido totalmente pela obrigatoriedade extrema da persistência permitir e aceitar o conceito da designação base por modelo primário explícito.
  - Para casos limitados de tópicos em logs sequenciais imutáveis como em brokers como Kafka puros (sem a opção "idempotent producer" acionada), exige um esforço lateral de compactação constante para deitar fora chaves redundantes e contorna a fragilidade temporária.