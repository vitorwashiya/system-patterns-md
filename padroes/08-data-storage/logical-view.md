# Padrão: Visão Lógica (Logical View)

## 1. Resumo (O que é?)
A Visão Lógica é um padrão de fachada analítica (*facade*) que encapsula complexidades ou unificações SQL gigantes e as expõe como se fossem uma tabela comum, pura e simples para o consumidor, mas que sob o capô recalcula os dados dinamicamente a cada requisição sem salvar arquivos extras.

## 2. O Problema
* A equipe criou a modelagem base. O time de negócios precisa da métrica de Vendas Líquidas que cruza 5 tabelas (Vendas, Descontos, Impostos, Fretes e Devoluções). 
* Se você entregar as 5 tabelas para os analistas, toda vez que o negócio pedir uma planilha, o analista da equipe A escreverá as regras matemáticas de desconto diferentes do analista B, gerando números corporativos inconsistentes. 
* Se você criar uma nova "Tabela de Vendas Líquidas", acabará pagando o dobro do armazenamento do servidor porque terá cópias de tudo.

## 3. A Solução
Esconda o SQL imenso numa `VIEW` Lógica. A Visão não tem memória e não salva arquivos físicos no HD. Ela atua apenas como uma definição salva da regra de negócio (uma espécie de *Macro* oficial). Quando o usuário digita `SELECT * FROM visao_vendas_liquidas`, o motor do banco transforma e empacota a consulta secreta subjacente junto com a pesquisa do usuário em tempo real contra as tabelas base brutas.

## 4. Consequências e Trade-offs
* **Vantagens:** Centraliza a governança dos cálculos corporativos (Fonte Única da Verdade), evita duplicação de dados caros, e garante que a abstração final sempre retorne resultados da versão e do segundo em que foi executada (*Live Data*).
* **Desvantagens/Atenção:** 
  * **Lentidão Linear (Sem Cache):** Como Views Lógicas nunca gravam o resultado, se o seu dashboard analítico puxa os dados do painel de 5 em 5 segundos, o banco será forçado a refazer a matemática pesada cruzando as 5 tabelas em *Loop* infinito. Seu custo computacional explodirá.
  * **Téia de Aranha (View on Views):** Views criadas em cima de outras Views viram labirintos. Como o banco compila a lógica inteira debaixo dos panos, o analista faz 1 query inocente, e sem saber aciona o banco para escrutinar 5 mil relatórios gerenciais e devorar os CPUs centrais num estrangulamento sistêmico fatal.

## 5. Exemplo de Aplicação Prática
Para ocultar colunas financeiras e garantir clareza, os engenheiros de Analytics (AEs) criam dezenas de tabelas-fantasma (Views) com a ferramenta `dbt` na camada `Gold` (Datamarts). Os cientistas trabalham em cima da Tabela `Ouro`, alheios aos JOINs complexos com a camada Prata e Bronze que permeiam a modelagem escondida da visualização.

## 6. Exemplo Simples de Código
```sql
-- Criando uma View abstrata sem duplicar o tamanho da tabela clientes
CREATE VIEW view_clientes_ativos_sudeste AS 
SELECT id, nome FROM clientes 
WHERE status = 'ATIVO' AND regiao IN ('SP', 'RJ', 'MG', 'ES');

-- O usuário usa como Tabela comum e pega os dados ao vivo:
SELECT * FROM view_clientes_ativos_sudeste;
```

## 7. Padrões Relacionados ou Nomes Similares
Contrapõe a utilidade de cache oferecida primariamente pela *Materialized View* e atua como pilar de apresentação, parecido com a filosofia lógica do *Dataset Materializer*.
