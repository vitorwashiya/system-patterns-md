# Padrão: Carga Completa (Full Loader)

## 1. Resumo (O que é?)
O padrão de Carga Completa consiste em ingerir um conjunto de dados inteiro a cada execução, em vez de processar apenas as partes novas ou alteradas. É uma abordagem direta e simples para transferir dados de uma origem para um destino.

Este padrão é muito útil para inicializar um banco de dados (bootstrap) ou para gerar conjuntos de dados de referência que mudam pouco com o tempo. Imagine copiar uma tabela inteira de um sistema antigo para um novo todos os dias.

## 2. O Problema
* Você precisa ingerir dados de um provedor externo que não possui uma coluna ou atributo indicando quais registros foram atualizados ou inseridos desde a última extração (como uma data de atualização).
* O conjunto de dados é pequeno ou evolui de forma muito lenta, não justificando a complexidade de uma lógica de extração incremental.

## 3. A Solução
A solução ideal é utilizar o padrão de Carga Completa. A implementação mais simples usa apenas duas etapas: Extração e Carga (Extract and Load - EL), usando comandos nativos do banco de dados para exportar e importar os dados sem transformações. 

Se os bancos de dados forem heterogêneos, a solução passa a ser um processo de Extração, Transformação e Carga (ETL), adicionando uma camada fina de transformação para adaptar os formatos de entrada e saída.

## 4. Consequências e Trade-offs
* **Vantagens:** Simplicidade de implementação; não requer atributos de controle (como datas de atualização) na origem.
* **Desvantagens/Atenção:** 
  * **Volume de Dados:** Requer tempo e recursos de processamento constantes, o que pode se tornar um gargalo ou causar falhas se o volume de dados crescer drasticamente de forma inesperada.
  * **Consistência dos Dados:** Se você usar a estratégia de apagar e inserir (drop-and-insert), consumidores podem ler dados parciais durante a execução. O ideal é usar transações ou criar "views" para alternar os dados de forma transparente.

## 5. Exemplo de Aplicação Prática
Um sistema precisa carregar diariamente uma lista de códigos e nomes de países a partir de uma API do governo que fornece sempre a lista completa. Como a lista é pequena e raramente muda, um processo roda toda madrugada, baixa a lista inteira e substitui a tabela atual do data warehouse.

## 6. Exemplo Simples de Código
```python
# Usando PySpark para ler dados completos e sobrescrever a tabela destino
dados_completos = spark.read.json("s3://origem/paises.json")

# A opção 'overwrite' garante a substituição completa dos dados
dados_completos.write.mode('overwrite').format('delta').save("s3://destino/paises")
```

## 7. Padrões Relacionados ou Nomes Similares
Também conhecido como *Passthrough Jobs* (quando não há transformação), *EL* (Extract, Load) e *ETL* (Extract, Transform, Load). Relaciona-se com padrões de sobreposição como *Data Overwrite* e *Fast Metadata Cleaner*.
