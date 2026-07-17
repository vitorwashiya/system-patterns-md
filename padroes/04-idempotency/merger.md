# Padrão: Mesclador (Merger)

## 1. Resumo (O que é?)
O padrão Mesclador (Merger) lida com atualizações de dados. Ele permite injetar conjuntos de dados pequenos e incrementais (com inserções, atualizações e exclusões lógicas) em um conjunto de dados alvo maior, operando como o tradicional UPSERT (Update + Insert).

## 2. O Problema
* O seu provedor manda apenas o que mudou na base dele no último dia. Se você usar estratégias de Sobrescrita Total (*Overwrite*), perderá todo o histórico de dados que não sofreu modificação.
* Você precisa sincronizar de forma exata e contínua essas mudanças com sua base alvo, sem gerar duplicação de dados, mesmo em caso de falhas ou re-execuções (*retries*).

## 3. A Solução
Use a funcionalidade nativa do banco de dados ou do formato de armazenamento chamada `MERGE` (ou comandos análogos como `UPSERT`). Primeiro, deve-se determinar quais colunas formam a chave primária de identidade única do registro (ex: `user_id`). A operação de merge comparará os dados recebidos com os da base principal. Se houver correspondência, ocorre o `UPDATE`; se não houver, ocorre o `INSERT`. Exclusões são feitas através do mapeamento de flags (soft deletes) no conjunto de dados entrante.

## 4. Consequências e Trade-offs
* **Vantagens:** Ideal para trabalhar com dados incrementais, evitando processar e sobrescrever tabelas gigantes inteiras.
* **Desvantagens/Atenção:** 
  * **Exige Chaves Únicas Claras:** Se a fonte de dados não fornecer identificadores únicos estáticos e imutáveis, o padrão não fará o cruzamento corretamente e duplicará os registros no backfilling.
  * **Custo de I/O:** É mais intenso computacionalmente que uma inserção pura, pois o mecanismo precisa ler ou usar as estatísticas do destino para realizar as comparações de "MATCH" antes de escrever.

## 5. Exemplo de Aplicação Prática
Um sistema recebe de hora em hora alterações de carrinho de compras dos clientes. Se o cliente 101 já tem carrinho salvo, a quantidade de itens na base analítica é atualizada (Update). Se é o primeiro clique do cliente 102, cria-se o registro no destino (Insert). Se o cliente 101 abandona e deleta o carrinho, o registro recebido vem com a flag `deletado=true`, o que induz a uma exclusão no destino.

## 6. Exemplo Simples de Código
```sql
-- Padrão MERGE (UPSERT) em bancos de dados relacionais e em Delta Lake
MERGE INTO clientes_destino AS t
USING clientes_atualizados AS f
ON t.id_cliente = f.id_cliente

WHEN MATCHED AND f.foi_deletado = true THEN DELETE
WHEN MATCHED THEN UPDATE SET t.nome = f.nome
WHEN NOT MATCHED THEN INSERT (id_cliente, nome) VALUES (f.id_cliente, f.nome)
```

## 7. Padrões Relacionados ou Nomes Similares
Popularmente conhecido como *UPSERT*. Relaciona-se ao *Stateful Merger* (para cobrir deficiências de consistência em backfillings) e muito utilizado como destino nos padrões de *Change Data Capture (CDC)*.
