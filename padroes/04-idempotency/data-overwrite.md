# Padrão: Sobrescrita de Dados (Data Overwrite)

## 1. Resumo (O que é?)
O padrão Sobrescrita de Dados visa garantir a idempotência de um pipeline através da completa substituição (sobrescrita) física dos dados existentes no destino. Quando a operação de metadados não está disponível ou é muito complexa, você substitui diretamente os blocos de arquivos por novas versões.

## 2. O Problema
* O pipeline grava dados em formatos de arquivo brutos no Object Storage (ex: S3/GCS), onde comandos de DDL (`TRUNCATE`, `DROP`) não funcionam diretamente.
* O fluxo atual gera dados duplicados sempre que é re-executado porque as novas execuções simplesmente anexam (`append`) novos dados aos que já foram salvos.

## 3. A Solução
Substitua inteiramente os arquivos ou a tabela no destino usando comandos ou configurações nativas dos frameworks de processamento voltadas para a "sobrescrita". Ao usar opções como `.mode('overwrite')` no Apache Spark ou comandos de banco como `INSERT OVERWRITE`, a própria ferramenta ou banco cuidará de excluir os arquivos ou diretórios antigos no caminho do destino, antes de colocar os novos dados.

## 4. Consequências e Trade-offs
* **Vantagens:** Abordagem muito universal, simples de implementar e que não requer camadas de gerenciamento de tabelas ou catálogos complexos.
* **Desvantagens/Atenção:** 
  * **Overhead de Dados:** Reprocessar grandes quantidades de dados sem estar propriamente particionado pode ser excessivamente caro e demorado, uma vez que requer leitura e regravação física total.
  * **Necessidade de Vacuum/Limpeza:** Em formatos de tabela modernos (como Delta Lake ou Iceberg), o overwrite cria uma nova versão na timeline de commits, mas as linhas mortas e arquivos velhos continuam ocultos no disco. Limpezas (*vacuum*) regulares são exigidas para salvar dinheiro.

## 5. Exemplo de Aplicação Prática
Todo dia um job extrai a lista de catálogo de clientes e seus endereços a partir de um sistema legado, gerando um CSV que reflete o estado atual inteiro. Esse CSV é depositado em um bucket AWS S3 na pasta `/clientes_atuais/`, e o próprio comando do utilitário garante a exclusão dos arquivos antigos nessa mesma pasta de forma atômica.

## 6. Exemplo Simples de Código
```python
# Utilizando o modo overwrite no PySpark
(
    dados_processados
    .write
    .mode('overwrite') # Assegura que toda a pasta ou tabela seja reescrita
    .format('parquet')
    .save('s3://meu-bucket/dados/clientes/')
)
```

## 7. Padrões Relacionados ou Nomes Similares
Também é uma estratégia de Idempotência. Concorrente direto do *Fast Metadata Cleaner*, sendo a abordagem preferida em casos de Data Lakes com formatos abertos ou ausência de orquestração de bancos relacionais.
