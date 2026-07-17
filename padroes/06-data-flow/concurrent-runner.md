# Padrão: Executor Concorrente (Concurrent Runner)

## 1. Resumo (O que é?)
O padrão de Executor Concorrente (*Concurrent Runner*) libera propositalmente a capacidade dos orquestradores de lançar e rodar múltiplas instâncias exatas (ou fatias de tempo) do mesmo pipeline de processamento em paralelo. 

## 2. O Problema
* Seu time é focado apenas na extração (sem estado dependente) e o foco é descer dados da API para o banco o mais rápido possível (a cada 30 minutos).
* Às vezes, o tráfego de rede engarrafa, a API demora, e o pipeline leva 45 minutos. Sob o padrão *Single Runner*, a extração das 14h ficaria travada até as 14h15, empurrando as demais. Você sabe que os blocos de leitura não dependem um do outro e quer tirar a amarra do tempo de execução sequencial.

## 3. A Solução
Relaxar a restrição de simultaneidade! Mude a capacidade de tarefas ativas concorrentes (`concurrency` ou `max_active_runs`) do seu orquestrador de 1 para um número razoável (por exemplo, 5 ou 100). Desde que seus repositórios aceitem cargas concorrentes sem *locks* no banco de dados, o orquestrador apenas agendará as iterações sem aguardar o sinal de liberação uns dos outros.

## 4. Consequências e Trade-offs
* **Vantagens:** Aborda e elimina os temíveis engarrafamentos cronológicos e resolve magistralmente a velocidade e agilidade exigidas por cenários de reprocessamento (Backfilling).
* **Desvantagens/Atenção:** 
  * **Inanição de Recursos (Resource Starvation):** Configurar execuções paralelas para 100 em um ambiente compartilhado (*multitenant*) pode usar toda a cota das máquinas (Workers) num Backfilling imenso, impedindo que outros pipelines críticos de outros departamentos rodem. Para resolver, use *Pools* de limitação no orquestrador.
  * **Conflitos de Estado Compartilhado (Shared State):** Em um processamento com estado onde a "Data de Última Atualização" é lida numa tabela controle, a concorrência sem trava cega a pipeline e gera duplicação pesada ou anulações no reprocessamento mútuo (Ver *Dynamic Late Data Integrator*).

## 5. Exemplo de Aplicação Prática
Para analisar o balanço de 10 anos de vendas (backfilling massivo mensal), você joga o script que rodaria uma vez por vez num Concurrent Runner. Imediatamente o Airflow despacha 120 trabalhos pros Workers da Nuvem em paralelo. Um processamento que no *Single Runner* levaria meses para completar a varredura acaba em poucos minutos.

## 6. Exemplo Simples de Código
```python
# Aumentando execuções simultâneas em Apache Airflow
with DAG(
    'extrator_livre_concorrencia',
    # Permite rodar até 5 períodos diferentes de extração juntos
    max_active_runs=5,
    default_args={
        # Ignora se o passado quebrou ou está atrasado
        'depends_on_past': False,
    }
) as dag:
    # ... processamento corre fluidamente
```

## 7. Padrões Relacionados ou Nomes Similares
Uma quebra das regras do *Single Runner*. Encontra amparo sistêmico muito forte com integrações nas tecnologias Serverless autossustentáveis (como GCP Dataflow ou AWS Step Functions) que suportam execuções por milhares.
