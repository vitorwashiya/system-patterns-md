# Padrão: Executor Único (Single Runner)

## 1. Resumo (O que é?)
O padrão de Executor Único (*Single Runner*) atua restringindo e garantindo que, independentemente do agendamento ou dos eventos de entrada, apenas uma única instância (execução) daquele pipeline rode por vez. 

## 2. O Problema
* O seu pipeline recém-construído (uma Prova de Conceito - POC) usava um *Incremental Sessionizer*, o que o obriga a executar as cargas de forma estritamente sequencial (uma iteração do job depende do estado final salvo na iteração prévia).
* Agora que a POC vai para produção e rodará automatizada via orquestrador, se duas horas tentarem rodar juntas, a concorrência causará duplicação e sobrescrita de estados incorretos.

## 3. A Solução
Para garantir a execução linear, impomos as regras de orquestração do Executor Único. O padrão restringe as configurações do motor do orquestrador (como no Apache Airflow ou AWS Step Functions) definindo rigidamente a concorrência global daquele *Job* para o limite máximo de 1. Além de restringir o paralelismo, ele força que execuções em espera não comecem até a antecedente obter o status finalizado de "Sucesso". Se a anterior falhar, as próximas não rodam (trancamento de fluxo).

## 4. Consequências e Trade-offs
* **Vantagens:** O único formato capaz de proteger de fato processamentos incrementais baseados no estado passado ou em versionamentos rigorosos sequenciais.
* **Desvantagens/Atenção:** 
  * **Problemas com Backfilling:** Recalcular um ano de dados processando uma tarefa após a outra (sequencialmente, 1 por 1) é extremamente lento. Se a regra exige paralelismo restrito, você perderá a vantagem da Nuvem elástica e pagará pelo pior tempo possível.
  * **Latência em Estrangulamento (Stragglers):** Se seu job é agendado a cada 1 hora, leva naturalmente 30 min, mas de repente um desvio nos dados faz ele levar 1.5 horas, o próximo job marcado não iniciará. Em seguida, os jobs vão empilhando, atrasando todo o faturamento ou geração de visão para a empresa (Gargalo Progressivo).

## 5. Exemplo de Aplicação Prática
Um robô notificador diário aponta falhas bancárias. A Diretoria pediu para que as lógicas nunca se sobreponham caso um erro pare o servidor. Logo, nas sextas-feiras de pico, a pipeline de 12:00h demora, e a pipeline agendada de 13:00h espera no status *Queued* até que a anterior finalize, preservando a emissão única.

## 6. Exemplo Simples de Código
```python
# Limitando a concorrência no Apache Airflow com depends_on_past
with DAG(
    'pipeline_executor_unico',
    # Limita o número de dags do mesmo tipo simultâneas a 1
    max_active_runs=1,
    default_args={
        # Trava tarefas pendentes caso a do período antecedente não tenha vingado
        'depends_on_past': True,
    }
) as dag:
    # ... as tarefas correm sob este contexto bloqueante
```

## 7. Padrões Relacionados ou Nomes Similares
Uma evolução global para a trava imposta pelo *Readiness Marker* em dados. Contrastado de forma antagônica pelas permissões liberais exigidas pelo *Concurrent Runner*.
