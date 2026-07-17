# Padrão: Carga Incremental (Incremental Loader)

## 1. Resumo (O que é?)
A Carga Incremental é um padrão de ingestão que processa apenas os registros novos ou modificados desde a última execução da tarefa. Ao invés de mover o conjunto de dados completo, ele captura apenas um subconjunto.

Esta abordagem é ideal para lidar com tabelas ou arquivos que crescem continuamente, pois economiza tempo e recursos de processamento ao evitar o retrabalho em dados já ingeridos.

## 2. O Problema
* O volume de dados de origem cresce de forma constante, tornando inviável e demorado realizar uma Carga Completa diária.
* Você precisa extrair eventos que chegam com frequência (como visitas a um site), onde apenas os novos registros inseridos após a última ingestão importam.

## 3. A Solução
O padrão Carga Incremental processa apenas os dados mais recentes. Isso geralmente é feito de duas formas:
1. Usando uma **coluna delta** (como data de criação ou atualização) para identificar e filtrar as linhas que são mais recentes que o momento da última execução.
2. Baseando-se em **dados particionados por tempo**, onde a ingestão carrega pastas ou partições completas correspondentes a uma hora ou dia específico, simplificando a filtragem de dados novos.

## 4. Consequências e Trade-offs
* **Vantagens:** Reduz drasticamente o volume de dados transferidos e o tempo de execução, permitindo ingestões mais frequentes.
* **Desvantagens/Atenção:** 
  * **Exclusões físicas (Hard Deletes):** Se o produtor excluir um registro na origem, a carga baseada em coluna delta não detectará essa exclusão. É necessário implementar "exclusões lógicas" (soft deletes) na origem.
  * **Reprocessamento (Backfilling):** Se você precisar reprocessar um longo período de tempo, o volume de dados pode se comportar como uma carga completa. É recomendável limitar o tamanho da janela de ingestão por execução.

## 5. Exemplo de Aplicação Prática
Um pipeline roda a cada hora para capturar novos registros de transações de vendas. Ele verifica qual foi o horário da última execução bem-sucedida e busca no banco de origem apenas as vendas cuja `data_venda` seja maior que a data registrada.

## 6. Exemplo Simples de Código
```python
# Lógica usando uma coluna delta para filtrar registros recentes
data_ultima_execucao = "2024-01-01 10:00:00"

dados_novos = spark.read.table("origem.vendas").filter(
    f"data_venda > '{data_ultima_execucao}'"
)

dados_novos.write.mode("append").format("delta").save("destino.vendas")
```

## 7. Padrões Relacionados ou Nomes Similares
Também chamado de *Delta Load* ou processamento por *Micro-batches*. Trabalha bem em conjunto com o padrão *Readiness Marker* para garantir que a partição de tempo está pronta para ser lida.
