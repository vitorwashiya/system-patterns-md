# Padrão: Tabela Interna (Internal Table / Managed Table)

## 1. Resumo (O que é?)
Ao contrário da Tabela Externa, a Tabela Interna (ou Tabela Gerenciada) entrega toda a posse, controle físico dos arquivos e estrutura para o sistema do banco de dados analítico. O banco dita a vida útil do arquivo e as organizações de otimização física.

## 2. O Problema
* A equipe criou centenas de Tabelas Externas soltas em formatos básicos. Mas o tempo de consulta de comandos analíticos está levando longos minutos, pois os arquivos foram salvos de modo caótico sem estratégias avançadas de ponteiros e índices.
* A equipe de engenharia não tem desenvolvedores focados para criar rotinas massivas de limpeza e empacotamento diário para otimizar os arquivos abertos.

## 3. A Solução
Utilizar a Tabela Interna nativa dos modernos Data Warehouses (como o BigQuery Storage nativo ou Snowflake) ou de formatos de lago modernos (Delta Tables em modo gerenciado no Databricks). Quando você carrega os dados para este repositório fechado, o serviço do banco não apenas aceita, mas reformata o dado fisicamente a portas fechadas. Ele cria metadados microscópicos, micro-partições ocultas e assume as responsabilidades vitais de manutenção (ex: rodar processos de `VACUUM`, `OPTIMIZE` e `Z-ORDERING`) automaticamente no *background* com o próprio motor embutido, cobrando o custo transparente na sua fatura da nuvem.

## 4. Consequências e Trade-offs
* **Vantagens:** Performance insana. Motores rodam dezenas de vezes mais rápido nessas estruturas fechadas pré-otimizadas do que buscando arquivos de texto cru no S3; zero manutenção administrativa da equipe de dados.
* **Desvantagens/Atenção:** 
  * **Refém do Fornecedor (Vendor Lock-in):** Como o banco assumiu os arquivos e os gravou numa estrutura proprietária, se você cancelar a assinatura com o provedor, todo aquele lago de dados analítico se torna irrecuperável por outras ferramentas open-source, exigindo exportações longas antes da migração.
  * **Destruição Absoluta (Drop Cascading):** Num descuido gravíssimo, se um engenheiro Júnior enviar um comando de teste apagando a tabela no console (`DROP TABLE vendas`), os arquivos físicos que pertenciam ao motor serão vaporizados da existência em conjunto, sem misericórdia, apagando a história da empresa (diferente da Tabela Externa, que apenas apaga a casca e preserva os arquivos).

## 5. Exemplo de Aplicação Prática
O Snowflake consome dezenas de CSVs cruzeiros e os internaliza em tabelas gerenciadas. Os arquivos não podem ser localizados abrindo pastas como no Windows, mas quando o analista pede pra sumarizar "O Total Gasto no Mês", o Snowflake responde em 2 segundos sem que o usuário entenda que por trás os dados estavam altamente zipados numa matriz colunar criptografada proprietária.

## 6. Exemplo Simples de Código
```sql
-- Criando (ou internalizando) no Delta Lake
-- Os dados e a manutenção passam a ser propriedade do ecossistema Delta.
CREATE TABLE vendas_internas
USING DELTA
AS SELECT * FROM dados_fonte;
```

## 7. Padrões Relacionados ou Nomes Similares
Geralmente referenciado como *Managed Table*. Tem conflitos técnicos e de filosofia com a *External Table* governada pelo usuário.
