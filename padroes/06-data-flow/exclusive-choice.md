# Padrão: Escolha Exclusiva (Exclusive Choice)

## 1. Resumo (O que é?)
O padrão de Escolha Exclusiva (*Exclusive Choice*) é o segundo padrão de bifurcação (Fan-Out). Diferente do *Parallel Split*, que executa tudo, este padrão conta com um roteador condicional que avalia regras e decide executar *apenas uma* das muitas ramificações descendentes disponíveis, ignorando o resto.

## 2. O Problema
* Você criou uma nova versão do seu job de ingestão de dados que começará a valer apenas a partir do dia 1 de Janeiro de 2024.
* Mas em casos de *backfilling* (reprocessamento de datas passadas), você quer que as datas anteriores a Janeiro/2024 continuem usando o job e a lógica antigos. 
* Você não quer criar dois pipelines separados para manter o histórico de execuções no mesmo painel de orquestração.

## 3. A Solução
Para rotear a execução de forma elegante, utiliza-se a Escolha Exclusiva. A implementação insere uma nova tarefa avaliadora de condições (um *branch operator*) logo antes das bifurcações. Essa tarefa analisa o contexto da execução atual (como a data da extração, ou os metadados do arquivo lido) e retorna qual caminho seguir. Todos os orquestradores modernos possuem suporte a isso nativamente (como o `BranchPythonOperator` do Airflow, ou atividades `If Condition` no Azure Data Factory).

## 4. Consequências e Trade-offs
* **Vantagens:** Flexibilidade imensa de regras de negócio; unifica fluxos diferentes mas correlatos (como lógica legado vs lógica nova) num lugar só.
* **Desvantagens/Atenção:** 
  * **Fábrica de Complexidade:** Ferramentas que abusam de if-elses visuais tendem a formar um "macarrão" incompreensível no orquestrador. Se você não consegue explicar para um colega por que aquele "if" existe rapidamente, a pipeline está complexa demais.
  * **Lógica Oculta (Hidden Logic):** Quando o padrão é implementado internamente dentro da camada de processamento (um `if/else` solto no script Python) em vez da camada de orquestração (visualmente no Airflow), as consequências tornam-se opacas e a equipe perderá a noção de quantas ramificações aquele código tem sem ler o código profundamente.
  * **Condições Pesadas:** Se o seu `if` precisar rodar uma *query* inteira nos dados pesados só para tomar a decisão, a performance do roteador destruirá o tempo do pipeline. Tente basear decisões sempre em metadados levinhos.

## 5. Exemplo de Aplicação Prática
Um pipeline consolida relatórios. A regra é: Se for o último dia do mês, ele segue pelo caminho da Direita e roda o Job de Fechamento Contábil Mensal. Se não for, ele ignora a direita, vai para o caminho da Esquerda e gera apenas o report Diário comum.

## 6. Exemplo Simples de Código
```python
# Utilizando o Operador de Branch do Apache Airflow
def decidir_qual_job_rodar(**context):
    data_migracao = pendulum.datetime(2024, 1, 1)
    if context['execution_date'] >= data_migracao:
        return 'rodar_job_novo'
    else:
        return 'rodar_job_legado'

branch_router = BranchPythonOperator(
    task_id='roteador_de_execucao',
    python_callable=decidir_qual_job_rodar
)

# O Roteador vai escolher executar apenas 1 desses dois caminhos
branch_router >> [rodar_job_novo, rodar_job_legado]
```

## 7. Padrões Relacionados ou Nomes Similares
Uma evolução ramificada do *Parallel Split*. Utiliza frequentemente os *Design Patterns* clássicos de Engenharia de Software como o *Factory Pattern* quando codificado no nível da aplicação.
