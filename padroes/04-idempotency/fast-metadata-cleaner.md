# Padrão: Limpador Rápido de Metadados (Fast Metadata Cleaner)

## 1. Resumo (O que é?)
O Limpador Rápido de Metadados é um padrão voltado para remover dados em grande escala de forma eficiente (idempotência por substituição). Em vez de tentar apagar linha por linha lendo os arquivos de dados, ele usa operações em nível de metadados para "esquecer" ou destruir logicamente o local de armazenamento (como jogar fora um fichário inteiro ao invés de apagar página por página).

## 2. O Problema
* O seu pipeline em lote precisa sobrescrever um conjunto de dados muito grande para garantir a idempotência. 
* Tentar remover milhares ou milhões de linhas com a instrução `DELETE` demora muito porque o banco de dados precisa identificar os registros, varrer os arquivos físicos, e então reescrevê-los. A performance está degradando diariamente.

## 3. A Solução
Use comandos como `TRUNCATE TABLE` ou `DROP TABLE` em partições de tabelas ou tabelas completas. Como eles agem sobre os metadados (o catálogo do banco) sem varrer as linhas físicas uma por uma, são quase instantâneos. O padrão geralmente exige que seus dados estejam segmentados fisicamente de maneira inteligente (ex: particionados por semana ou dia), para que você possa destruir e recriar rapidamente apenas a partição ou tabela específica que está processando no momento.

## 4. Consequências e Trade-offs
* **Vantagens:** É extremamente rápido em comparação com `DELETE`, tornando-se muito útil para limpezas brutais de dados grandes em pipelines idempotentes.
* **Desvantagens/Atenção:** 
  * **Granularidade do Backfilling:** Se você particionou a tabela por semana, não tem como limpar apenas um dia problemático. A unidade mínima de limpeza será uma semana inteira, forçando o reprocessamento de mais dias.
  * **Limites de Metadados:** Se você usar partições granulares (como de hora em hora) ou milhares de tabelas menores, pode atingir rapidamente o limite de partições do seu Data Warehouse (ex: BigQuery tem limite de 4.000 partições por tabela).

## 5. Exemplo de Aplicação Prática
Você possui um modelo de dados em que os dados de cada semana são mantidos em uma tabela física separada, mas todas as semanas juntas alimentam uma *View* (visualização) unificada. Ao processar os dados da nova semana, o pipeline primeiro emite um `DROP TABLE` na tabela daquela semana atual (limpando qualquer falha anterior instantaneamente) e insere os dados da extração de forma segura.

## 6. Exemplo Simples de Código
```sql
-- Em um banco de dados relacional ou data warehouse
-- Limpando toda a partição ou tabela instantaneamente via metadados
TRUNCATE TABLE visitas_semana_42;

-- Seguida da inserção segura dos dados calculados
INSERT INTO visitas_semana_42 SELECT * FROM visitas_brutas;
```

## 7. Padrões Relacionados ou Nomes Similares
Usado como estratégia de Idempotência baseada em *Overwriting* (Sobrescrita). Contrapõe-se ao padrão *Data Overwrite* (Sobrescrita de Dados) que lida com dados físicos ao invés de puramente metadados.
