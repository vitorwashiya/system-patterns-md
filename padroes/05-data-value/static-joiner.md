# Padrão: Junção Estática (Static Joiner)

## 1. Resumo (O que é?)
A Junção Estática (Static Joiner) é um padrão de enriquecimento de dados em que um fluxo de dados contínuo (ou lote) é combinado com um conjunto de dados de referência estático (ou de evolução lenta - SCD). O objetivo é adicionar mais contexto aos dados puros.

## 2. O Problema
* Os dados brutos gerados pela sua aplicação contêm apenas IDs (como `user_id`), mas não têm o contexto completo (como o perfil do usuário ou a data de registro).
* Consultas analíticas precisam dessas informações juntas para extrair valor de negócio.

## 3. A Solução
O padrão combina o conjunto de dados base com a tabela estática usando uma chave comum (ex: `user_id`). Para tabelas que sofrem atualizações lentas (Slowly Changing Dimensions - SCD), a junção também pode exigir restrições de tempo para combinar o evento com o contexto exato daquele momento (ex: "Junte usando o ID e garanta que a data do evento está entre a data de início e fim daquele registro").

## 4. Consequências e Trade-offs
* **Vantagens:** Aumenta significativamente a utilidade do dado e evita que a lógica de "lookup" seja reescrita por todos os consumidores a jusante.
* **Desvantagens/Atenção:** 
  * **Atraso de Dados (Late Data):** Em streaming, a base de referência pode estar desatualizada em relação ao evento recebido.
  * **Idempotência:** A reconstrução histórica (backfilling) exige que o estado passado do banco estático possa ser consultado no momento exato (usando técnicas de Time Travel ou SCD tipo 2/4).

## 5. Exemplo de Aplicação Prática
Um evento de clique no site traz apenas o `product_id`. Durante a ingestão na camada "Silver", o job consulta uma tabela estática de "Produtos" (dimension table) e anexa a categoria e o preço ao evento do clique.

## 6. Exemplo Simples de Código
```sql
-- Exemplo de Junção com Slowly Changing Dimension (Tipo 2)
SELECT e.evento_id, e.horario, p.nome, p.categoria
FROM eventos e
JOIN produtos p ON e.produto_id = p.id
 AND e.horario BETWEEN p.data_inicio AND p.data_fim;
```

## 7. Padrões Relacionados ou Nomes Similares
Uma fundação para padrões como *Dynamic Joiner*. Lida muito com conceitos de *Slowly Changing Dimensions (SCD)*.
