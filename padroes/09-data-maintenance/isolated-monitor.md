# Padrão: Monitor Isolado (Isolated Monitor)

## 1. Resumo (O que é?)
O Monitor Isolado retira completamente a inteligência de monitoria e verificação de qualidade de dentro dos pipelines de processamento produtivo. Ele transfere essa carga para trabalhos (Jobs) paralelos e inteiramente desconectados do fluxo que movem os dados reais.

## 2. O Problema
* Vocês implementaram centenas de métricas de Qualidade nos pipelines produtivos usando *Quality Checker* e *Continuous Monitor*. 
* Agora a equipe de negócios pediu novas regras de verificação. Você precisa abrir o código crítico da pipeline que processa os pagamentos do Brasil, reescrever as checagens lá no meio, aprovar Pull Requests de código severo e arriscar quebrar um script frágil apenas para adicionar uma verificação analítica. O Acoplamento virou o problema.

## 3. A Solução
Construa pipelines de auditoria secundárias (Monitores Isolados). O fluxo 1 (Pipeline de Negócio) apenas extrai, limpa, cruza as contas bancárias e grava os dados massivos na Tabela `X`, da forma mais burra e rápida possível. 
Em seguida, um fluxo 2 (Pipeline de Auditoria), que roda em um agendamento completamente independente, conecta-se a essa Tabela `X` depois de ela já existir no banco de dados e executa as validações com SQL ou ferramentas de Data Profiling sobre o modelo "em descanso".

## 4. Consequências e Trade-offs
* **Vantagens:** Extrema autonomia para as equipes (O time de Governança pode codificar centenas de métricas, alertas e Dashboards diários sem mexer ou depender da equipe da Engenharia pesada de Pipeline). Elimina a lentidão agregada de monitoria nas janelas severas de processamento.
* **Desvantagens/Atenção:** 
  * **Problemas de Ponto-no-Tempo (Point-in-Time):** Como o monitor não viaja com o dado pelo pipeline, ele verifica a "tabela cheia". Se algo deu errado na partição de ontem, mas o script está lendo tudo sem critério, é muito mais complexo saber exatamente se foi um erro da ingestão de 5 minutos atrás ou lixo acumulado da semana toda.
  * **Zero Garantias de Consistência Imediata:** Permite absolutamente todos os defeitos de fluxo passarem por longas horas antes da auditoria ser agendada.

## 5. Exemplo de Aplicação Prática
Para centralizar as regras, a empresa usa a ferramenta corporativa `dbt` para rodar seus "dbt Tests". Em vez de misturar isso nos Jobs de Spark, todo dia de manhã, um DAG no Airflow chamado "Auditor_Corporativo" roda disparando testes nas 100 tabelas principais do banco Snowflake. Tabelas isoladas ganham alertas, tabelas perfeitas passam no painel, nada encosta no Spark.

## 6. Exemplo Simples de Código
```sql
-- Um script Dbt Test Isolado focado apenas na integridade Pós-Carga
-- Se essa query retornar linhas, significa que há problema na tabela. 
-- Mas ela roda independente do ETL que a gerou.
SELECT
    id_produto,
    COUNT(*) as duplicados
FROM
    catalogo_produtos
GROUP BY
    id_produto
HAVING COUNT(*) > 1;
```

## 7. Padrões Relacionados ou Nomes Similares
Este padrão desmembra a técnica englobada no *Continuous Monitor*, levando-a para uma dimensão de orquestração assíncrona semelhante à relação estabelecida no *Isolated Sequencer*.
