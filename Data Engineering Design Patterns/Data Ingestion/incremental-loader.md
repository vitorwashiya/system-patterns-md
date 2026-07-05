---
# Incremental Loader

## Classificação
- **Tipo:** Criacional
- **Escopo:** Objeto

## Intenção
Ingerir subconjuntos novos ou modificados de um sistema de origem de forma contínua, anexando estritamente os registros filtrados em um armazenamento destino com o intuito de otimizar volumes e custos.

## Também Conhecido Como
Incremental Load.

## Motivação (Contexto)
Extração de registros imutáveis de eventos orientados a transações legadas que crescem veloz e ininterruptamente, em cenários onde processamentos massivos de substituição completa incorrem em latências inviáveis e exaustão absurda de rede.

## Aplicabilidade
- Use o Incremental Loader quando:
  - O conjunto de dados original aumenta fisicamente de forma contínua.
  - A fonte fornece colunas delimitadoras (delta columns) como timestamps consistentes ou partições baseadas primariamente em tempo.
  - Registros são preferencialmente imutáveis (append-only) ou excluem dados passivamente (soft deletes).

## Estrutura e Participantes
- **Data Provider:** Sistema de registro persistente com dados versionados por tempo ou estruturados por partições.
- **Incremental Job:** Processo de controle temporizado que avalia a diferença do estado operacional atual contra execuções passadas extraindo apenas a métrica de transação nova.
- **Append Storage:** Repositório destino estritamente tolerante à adição contínua (append-only).

## Colaborações
O Incremental Job pesquisa ciclicamente o instante lógico do último ingest (watermark state), invoca as consultas ao Data Provider exigindo dados gerados perfeitamente acima daquele timestamp delimitador, e consolida os adendos no Append Storage.

## Consequências
- **Prós:**
  - Minimização severa do tráfego nativo de rede e do dispêndio de recursos de processamento por ciclo ingestivo.
  - Habilita empiricamente agendamentos de extração analíticos agressivos e muito mais frequentes.
- **Contras:**
  - Exclusões físicas reais (hard deletes) somem misteriosamente da captura, provocando defasagem no repasse lógico se não operadas como tabelas insert-only.
  - Retentativas exaustivas massivas para repovoamento passado (backfilling) tendem a colapsar se a janela temporal não contiver limitações estruturais (Intervalos curtos obrigatórios).

## Implementação (Exemplo de Código)
```python
# Ingestão em Apache Spark fundamentada primariamente em Delta Column isolada
in_data = (spark_session.read.text(input_path)
           .select(functions.from_json(functions.col('value'), 'ingestion_time TIMESTAMP')))

# Filtra estritamente a janela temporal incremental atualizada
input_to_write = in_data.filter(
    f'ingestion_time BETWEEN "{date_from}" AND "{date_to}"'
)

# Anexação isolada puramente sequencial preservando eventos maduros no alvo final
input_to_write.write.mode('append').text(output_path)

# ---
# Alternativa: Ingestão baseada em partições físicas (Ex: Airflow + Spark)
# A partição de data é resolvida de forma declarativa e passada via macros do orquestrador (ex: {{ ds }})
execution_date = "2023-11-24" # Valor injetado dinamicamente

# Leitura direta do diretório particionado correspondente à janela de execução
in_data_partitioned = spark_session.read.format("parquet").load(f"/data/input/date={execution_date}")

# Como os dados já estão delimitados fisicamente, a gravação é isolada sem a necessidade do .filter()
in_data_partitioned.write.mode("append").format("parquet").save(f"/data/output/date={execution_date}")
```

## Padrões Relacionados

* **Change Data Capture:** Adotado enfaticamente nos impasses diretos em que se exige controle total transacional sobre hard deletes imediatos.
* **Readiness Marker:** Empregado nativamente em harmonia para consolidar partições orientadas a tempo esperando pela chancela de finalização integral.

---
