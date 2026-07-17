# Padrão: Baldeamento (Bucket / Bucketing)

## 1. Resumo (O que é?)
O padrão *Bucket* atua como um refinador complementar da organização física. Enquanto o *Horizontal Partitioner* agrupa dados idênticos (ex: Dia 01), o Bucketing distribui (Hash) e divide arquivos baseados numa chave de altíssima diversidade (Alta Cardinalidade, ex: ID do Usuário) em um número fixo de "baldes" limitados para otimizar *Joins* na rede.

## 2. O Problema
* O *Horizontal Partitioner* avisa para você não particionar dados por "ID do Cliente", porque você tem 50 milhões de clientes, o que criaria 50 milhões de micro-pastas e explodiria o disco.
* Mas você realmente precisa cruzar (JOIN) a tabela "Vendas" com "Perfis" pelo "ID do Cliente" diariamente. No padrão de junção distribuída, misturar e reorganizar esses milhões de usuários através da rede (Shuffle) arrasta a performance ao limite.

## 3. A Solução
Use o particionador de blocos: Bucketing. Você diz ao motor de processamento (ex: Apache Hive ou Spark) para espalhar os "IDs" dentro de, exatamente, `N` baldes (ex: 200). Ele usará uma função matemática (Hash) onde o Cliente João (`ID 5`) sempre será guardado no `Balde 10` e a Maria (`ID 88`) sempre cairá no `Balde 21`. Você faz o mesmo na tabela de Perfil. 
Como as duas tabelas possuem o mesmo número de gavetas matemáticas, na hora de fazer o `JOIN`, a máquina sabe que não precisa pedir partes do "João" pela rede; ela sabe empiricamente que as transações e o perfil de todo mundo do Balde 10 já moram juntos fisicamente no mesmo lugar! (Isso habilita o padrão *Local Aggregator/Joiner*).

## 4. Consequências e Trade-offs
* **Vantagens:** Otimiza drasticamente cruzamentos (*JOINs*) monstruosos, evitando embaralhamento na rede (*Sort-Merge Join* vira um Join Local direto). Impede o temido problema dos "Small Files" fixando o limite final em 200 baldes.
* **Desvantagens/Atenção:** 
  * **Imutabilidade Estrutural:** O arquivo não pode ser redimensionado levemente. Se você definir 200 buckets hoje e sua empresa crescer muito e precisar de 1000 buckets, você será obrigado a ler a base inteira, destruí-la e reescrevê-la desde 2010.
  * **Acoplamento Matemático:** A magia dos arquivos distribuídos requer sincronia perfeita. Você só ganhará benefícios reais se agrupar a Tabela B com a exata mesma quantidade de baldes (Ex: ambas 200) da Tabela A.

## 5. Exemplo de Aplicação Prática
Para juntar os impostos e as receitas diárias por usuário na Receita Federal, o modelo cria 1024 Buckets nos arquivos pela chave de `CPF`. No fim do dia, a união pesada entre bancos acontece nos próprios nós distribuídos isolados instantaneamente, visto que a infraestrutura pré-alocou os CPFs com finais iguais de ambas as tabelas nas mesas correspondentes.

## 6. Exemplo Simples de Código
```sql
-- Declarando Buckets num CREATE TABLE do Hive/Spark SQL
CREATE TABLE pedidos_clientes (
    cliente_id INT,
    valor DECIMAL
)
-- Cria 100 gavetas matemáticas perfeitas baseadas num Hash do cliente_id
CLUSTERED BY (cliente_id) INTO 100 BUCKETS
STORED AS PARQUET;
```

## 7. Padrões Relacionados ou Nomes Similares
Implementa fisicamente a base exigida para o funcionamento do padrão de processamento *Local Aggregator* e é um aliado do *Horizontal Partitioner*.
