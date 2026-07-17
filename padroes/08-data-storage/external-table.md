# Padrão: Tabela Externa (External Table)

## 1. Resumo (O que é?)
A Tabela Externa é uma abstração virtual conectada a um motor analítico, onde o sistema registra apenas os "metadados" (o formato da tabela: Nome, Colunas, Tipos), mas os dados reais (os arquivos físicos) permanecem hospedados do lado de fora, livres, dentro de um Object Storage genérico governado pelo usuário.

## 2. O Problema
* Você salvou milhares de planilhas Parquet no seu Data Lake (S3).
* Para que os analistas (que só sabem a linguagem SQL) consigam ler isso no BigQuery ou Amazon Athena, os dados teriam de ser sugados, formatados e hospedados dentro do próprio espaço físico e fechado do banco (criando dados duplicados e contas altíssimas de armazenamento premium).

## 3. A Solução
Deixe os arquivos soltos onde estão. Crie uma Tabela Externa no banco de dados. Essa tabela é apenas uma casca (um contrato): ela avisa ao banco "A Tabela Vendas tem 3 colunas e elas podem ser encontradas caso você vá ler os arquivos Parquet que estão lá na pasta `/data/vendas/` do S3". Quando o analista digita o `SELECT`, o banco de dados viaja até a pasta externa, escaneia os arquivos, projeta a visualização de tabela e exibe para o usuário em SQL. 

## 4. Consequências e Trade-offs
* **Vantagens:** O dado é seu e da sua empresa, não do banco de dados (sem amarras ao fornecedor). Permite que ferramentas abertas como Spark, Athena e Trino consultem os mesmos arquivos simultaneamente sem cópias ocultas.
* **Desvantagens/Atenção:** 
  * **Falta de Recursos DML em formatos simples:** Se os arquivos externos forem CSV ou JSON simples, comandos de atualização ou delete (`UPDATE`/`DELETE`) não funcionarão através da tabela externa, forçando a reescrita maciça. (Isto é resolvido adotando formatos de tabela abertos como Apache Iceberg/Delta, que implementam as *Internal Tables* em cima do armazenamento aberto).
  * **Lentidão por Escaneamento (Full Scan):** Consultar dados por tabelas externas sem as partições declaradas faz o banco ler o arquivo inteiro só pra achar 1 linha, aumentando brutalmente as taxas de cobrança por megabyte consultado de serviços serverless.

## 5. Exemplo de Aplicação Prática
Para democratizar análises, um engenheiro aponta o AWS Athena para a pasta S3 de Logs do mês. O analista comercial consegue agora usar o console do Athena (SQL) cruzando os logs diários de compras mesmo sendo um arquivo não relacional do S3. Ao deletar a tabela no Athena, os arquivos no S3 não sofrem nenhum dano e não somem.

## 6. Exemplo Simples de Código
```sql
-- Criando uma abstração na Hive/Athena sobre os arquivos S3 (Nenhum dado é importado)
CREATE EXTERNAL TABLE IF NOT EXISTS vendas_externas (
  id_venda INT,
  valor DECIMAL
)
STORED AS PARQUET
LOCATION 's3://meu-data-lake/vendas/';
```

## 7. Padrões Relacionados ou Nomes Similares
Uma progressão do uso inteligente de *Object Storage*. Contrapõe as lógicas proprietárias da *Internal Table*.
