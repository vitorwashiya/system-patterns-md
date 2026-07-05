
# Compactor

## Classificação
- **Tipo:** Estrutural
- **Escopo:** Objeto

## Intenção
Consolidar proativamente microarquivos fragmentados e diretórios minúsculos excessivos num repositório em blocos otimizados extensos, dirimindo o ônus exaustivo latente sobre as varreduras de sistemas de leitura analítica downstream.

## Também Conhecido Como
Merge on Read, Log Compaction, Otimização Ativa de Storage.

## Motivação (Contexto)
Lakehouses acoplados a ingestas de fluxos velozes e picotados (micro-batches / streams contínuos) sofrem do insidioso "metadata overhead" problem. Jobs analíticos de leitores começam a despender arbitrariamente até 70% da computação exclusivamente abrindo e listando arquivos insignificantes. O imperativo torna-se re-aglutinar silenciosamente estas cascatas em pedaços maduros unitários nos bastidores garantindo robustez paralela.

## Aplicabilidade
- Use o Compactor quando:
  - Consumidores experimentam lentidão e degradação crônica progressiva limitante originada da proliferação insana e exponencial de arquivos lidos em instâncias analíticas I/O.
  - Sistemas pautados estritamente pela indexação de chaves (Kafka, tópicos longos) padecem da inflação desnecessária mantendo mutações obsoletas presas no mesmo segmento restritivo de log.

## Estrutura e Participantes
- **Fragmented Storage:** O armazenamento nativo hospedando volumes excessivos insuportáveis de coleções microscópicas fracionadas.
- **Compactor Job (Optimizer):** Processo corretivo ativado de maneira esporádica off-peak varrendo, desfragmentando e escrevendo passivamente as entidades substitutivas otimizadas atômicas.

## Colaborações
Com cadência rigidamente espaçada, o Compactor Job examina metadados no Fragmented Storage. Intercepta blocos analíticos subjacentes combinando-os em matrizes maciças e homogêneas e efetiva novos commits de arquivos expandidos, desassociando magicamente as listagens obsoletas no transaction log transiente.

## Consequências
- **Prós:**
  - Otimização radical imperativa dos custos reais na varredura I/O analítica para ferramentas analíticas downstream.
  - Extirpa o volume de blocos inativos suprimidos reduntantemente da ocupação de disco em logs consolidados.
- **Contras:**
  - Conflito doloroso do "Trade-off Computacional": rotinas frequentes são estupendamente lentas e taxativas financeiramente onerando os orçamentos subjacentes da nuvem.
  - A operação em si deixa para trás vestígios órfãos e arquivos fantasma; requerimentos absolutos dependem da adição impositiva de jobs explícitos lixeiros de purga (VACUUM operations) acoplados.

## Implementação (Exemplo de Código)
```python
# Compactação utilizando API relacional de Otimização Delta Lake atômica
devices_table = DeltaTable.forPath(spark_session, table_dir)

# Aglutina fatias infinitesimais espalhadas de eventos reescrevendo em Parquets maciços e eficientes
devices_table.optimize().executeCompaction()

# Aviso: Purgações isoladas subjacentes de "Tombstones" órfãos restam pendentes neste estagio
# devices_table.vacuum()
```

## Padrões Relacionados

* **(Sem padrões relacionados definidos)** Na arquitetura de dados moderna, formatos de tabela abertos (Iceberg/Delta) carregam frequentemente esse padrão intrínseco.

---
