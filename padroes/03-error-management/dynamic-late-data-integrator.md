# Padrão: Integrador Dinâmico de Dados Atrasados (Dynamic Late Data Integrator)

## 1. Resumo (O que é?)
O Integrador Dinâmico de Dados Atrasados é uma evolução do modelo estático, criado para integrar de forma eficiente dados que chegam com muito atraso, sem processar desnecessariamente partições que não sofreram alterações. Ele lê ativamente o estado de modificação das partições para decidir o que precisa ser reprocessado.

## 2. O Problema
* A janela estática (Static Late Data Integrator) que você implementou não é suficiente, porque às vezes chegam dados com atrasos maiores que o limite configurado (ex: mais de 15 dias).
* Você quer aumentar a janela de tolerância para integrar esses dados muito antigos, mas reprocessar cegamente meses de dados todos os dias seria um grande desperdício de recursos e causaria alto custo.

## 3. A Solução
Para resolver isso, você deve criar e manter uma **Tabela de Estado (State Table)** que rastreia quando cada partição foi processada pela última vez pelo seu pipeline e qual foi o momento real de última atualização daquela partição na origem. 

O orquestrador fará uma consulta antes de iniciar as tarefas de integração de atraso: *"Quais partições tiveram sua data de modificação na origem alterada após a última vez que nós as processamos?"*. Apenas as partições que retornarem dessa consulta serão alvo do *backfilling* (reprocessamento dinâmico).

## 4. Consequências e Trade-offs
* **Vantagens:** Otimiza drasticamente o uso de recursos, pois só realiza o trabalho pesado onde realmente houve novos dados atrasados; flexibiliza a janela de *lookback*.
* **Desvantagens/Atenção:** 
  * **Problemas de Concorrência:** Se seu pipeline suportar execuções simultâneas, uma partição pode tentar ser integrada mais de uma vez ao mesmo tempo. É necessário travar (lock) o status da partição na tabela de estado durante o processamento para evitar corridas (race conditions).
  * **Depende de Metadados Claros:** Requer que o banco ou formato de arquivo (ex: Delta Lake, BigQuery) informe facilmente qual foi a última data de modificação da partição, senão você precisará codificar lógicas complexas de varredura.

## 5. Exemplo de Aplicação Prática
Seu pipeline preenche estatísticas financeiras que podem retroagir meses se um usuário contestar uma transação. Antes de iniciar, o fluxo consulta a tabela de metadados: "Alguma partição dos últimos 6 meses mudou hoje?". O banco responde que apenas a partição "2024-03-12" mudou. O sistema então reprocessa *apenas* essa partição específica, ignorando o resto.

## 6. Exemplo Simples de Código
```sql
-- Lógica que o orquestrador faria na tabela de Estado
SELECT nome_particao 
FROM tabela_estado_pipeline 
WHERE tempo_ultima_atualizacao_origem > tempo_ultimo_processamento_pipeline
  AND em_processamento = false; -- Prevenindo concorrência
```

## 7. Padrões Relacionados ou Nomes Similares
Uma variação otimizada do *Static Late Data Integrator*. Para gerenciar as execuções baseadas nessa tabela, frameworks usam recursos como o *Dynamic Task Mapping* (no Apache Airflow) para criar dinamicamente os trabalhos que faltam.
