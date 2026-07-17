# Padrão: Divisão Paralela (Parallel Split)

## 1. Resumo (O que é?)
A Divisão Paralela (*Parallel Split*) é um padrão da família *Fan-Out* em orquestração, no qual a conclusão ou exigência de uma única tarefa principal (a raiz/pai) dispara a execução de pelo menos duas (ou mais) ramificações descendentes simultaneamente, em rotas completamente isoladas.

## 2. O Problema
* Você precisa modernizar o processamento de dados substituindo um framework em C# por Python. A equipe migrou a infraestrutura, mas por medo de imperfeições sistêmicas, quer usar os dois frameworks rodando em produção até garantir validade por um mês.
* Como o resultado base das regras iniciais servirá tanto para gerar relatórios pela lógica do C# no banco legado, quanto para a lógica do Python em tabelas do Data Lake, você tem de direcionar o processamento idêntico em fluxos distintos.

## 3. A Solução
Este cenário focado em uma tarefa antecedente distribuindo saída requer a Divisão Paralela. Em orquestradores de dados de mercado (Airflow/Dagster) ou em programação simples na camada de execução (Spark), você cria os nós do grafo definindo a dependência no mesmo elemento pai. O essencial é garantir que a computação do pai (ex: a extração ou filtro inicial) não seja re-executada duplamente. Ferramentas que abstraem as dependências lidam com isso em *cache*, permitindo persistências eficientes na memória.

## 4. Consequências e Trade-offs
* **Vantagens:** Desacoplamento estrutural altíssimo; acelera a orquestração processando múltiplos caminhos analíticos concorrentes no mesmo tempo sem duplicação do esforço de leitura raiz.
* **Desvantagens/Atenção:** 
  * **Atrasos e Gargalos Dependentes (Blocked Execution):** Se o pipeline exige sequência baseada no término global da ramificação para iniciar no dia seguinte, a finalização dependerá da sua rota paralela mais lenta de todas.
  * **Problemas de Escalabilidade Física:** Se os fluxos em paralelo tiverem necessidades distintas de arquitetura (ex: um galho exigindo intenso CPU para ML, e outro de RAM pesada para JOINs tabulares), sua máquina que cuida do pai não comportará todas as filhas de maneira eficiente. Terá de recuar e usar *Local Sequencer* separando recursos.

## 5. Exemplo de Aplicação Prática
Os dados da campanha de TV são capturados por um job único às 03:00. O fechamento da extração desencadeia duas vias em paralelo: a Via A roda um script R para os Cientistas de Dados elaborarem previsão de churn e a Via B lança um script SQL focado na montagem das visões tabulares para planilhas da equipe de Marketing. Nenhuma atrasa a outra.

## 6. Exemplo Simples de Código
```python
# Exemplo de Divisão Paralela declarativa (Fan-Out)
carga_da_origem = FileSensor(task_id='esperar_arquivo_chegar')

job_transformar_para_lake = PythonOperator(task_id='job_lake_delta')
job_transformar_para_csv = PythonOperator(task_id='job_csv_legado')

# Um pai dividindo execução simultânea (Parallel Split)
carga_da_origem >> [job_transformar_para_lake, job_transformar_para_csv]
```

## 7. Padrões Relacionados ou Nomes Similares
Pertence à família *Fan-Out*. Apresenta paralelos técnicos opostos com os limites da *Exclusive Choice* (Escolha Exclusiva), na qual somente uma ramificação sobrevive.
