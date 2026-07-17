# Padrão: Visão Materializada (Materialized View)

## 1. Resumo (O que é?)
A Visão Materializada (*Materialized View*) é um objeto do banco de dados que preenche a lacuna de performance da Visão Lógica. Ela não apenas guarda as regras (Query SQL), mas efetivamente armazena o resultado final fisicamente (no disco, consumindo memória). Atua como um *cache* massivo para cálculos agressivamente lentos e repetitivos.

## 2. O Problema
* Os analistas adoraram que você empacotou as lógicas usando a *Logical View*. Mas a Visão consolida o balanço anual histórico global de todo o grupo (agrupando 10 bilhões de linhas).
* Toda manhã, 50 Diretores abrem o mesmo Dashboard no Tableau, gerando 50 execuções idênticas do cruzamento dos mesmos 10 bilhões de arquivos, causando lentidão insuportável no painel, bloqueando recursos e falindo o orçamento computacional.

## 3. A Solução
Altere a `VIEW` para `MATERIALIZED VIEW`. Nesta modalidade, o banco rodará aquela matemática terrível de 10 horas apenas uma única vez na surdina e tirará uma "fotografia" estática final da Tabela Resumo, gravando os blocos no disco. Agora, as 50 execuções do Tableau baterão na foto, retornando instantaneamente (milisegundos) com custo nulo de CPU analítico. E quando os dados novos chegarem? O banco utilizará inteligência e rotinas automatizadas para reescrever incrementalmente a fotografia de acordo com as mudanças (Refresh).

## 4. Consequências e Trade-offs
* **Vantagens:** Uma resposta suprema para performance de consultas repetitivas de Business Intelligence, abatendo consideravelmente o custo dinâmico com processamento em nuvem (*Compute Overload*).
* **Desvantagens/Atenção:** 
  * **O Atraso (Data Staleness):** As Materializações não são dados vivos puros; elas são atrasadas. Se a tabela-mãe receber Vendas 12:00, e a "foto" da *View* tiver regras para dar Refresh só 13:00, o CEO estará lendo os totais de forma parcial na reunião das 12:30.
  * **Complexidade de Atualização Incremental (Refresh):** Bancos modernos sabem analisar logs e atualizar apenas o que mudou na base raiz, mas dependendo das ferramentas ou lógicas matemáticas (`DISTINCT` complexos ou Window Functions), o motor falha no incremental e roda um "Full Refresh", regravando e processando a base inteira, o que pode consumir horas noturnas no gargalo.

## 5. Exemplo de Aplicação Prática
Para liberar os relatórios do aplicativo dos acionistas (com milhões de usuários puxando os saldos bancários parciais agrupados pelo último mês), o BigQuery materializa o SQL global todas as madrugadas. Durante o dia, todos os apps apontam velozmente pra Visão Materializada, que atua como um enorme espelho cache barato de armazenamento.

## 6. Exemplo Simples de Código
```sql
-- Criando Materialização em bancos como o Postgres ou Redshift
CREATE MATERIALIZED VIEW mv_vendas_por_cidade AS 
SELECT cidade, SUM(valor) FROM vendas GROUP BY cidade;

-- Processo imperativo exigindo atualização da "Fotografia" baseada no status das matrizes atuais
REFRESH MATERIALIZED VIEW mv_vendas_por_cidade;
```

## 7. Padrões Relacionados ou Nomes Similares
Uma progressão custosa, porém performática, da *Logical View*. Realiza internamente o conceito do que se espera manualmente construindo *Incremental Loaders* via pipelines externas.
