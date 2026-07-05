---
# 🧠 Cérebro do Agente: Roteador de Padrões de Projeto

## Instruções para a IA (System Prompting Integrado)
**ATENÇÃO AGENTE:** Leia a tabela de roteamento abaixo para iniciar seu raciocínio. Identifique qual padrão de projeto se alinha com o problema atual do usuário ou o contexto do código que você está manipulando. Uma vez identificado, você deve obrigatoriamente ler o arquivo `.md` correspondente na tabela usando a ferramenta de leitura de arquivos (como `view_file` ou equivalente) antes de planejar ou gerar qualquer código. Nunca tente adivinhar os detalhes de implementação, sintaxe ou arquitetura do padrão; sempre consulte o arquivo referenciado como sua fonte da verdade.

## Índice Semântico e Tabela de Roteamento

| Padrão | Quando usar (Gatilho Semântico) | Caminho do Arquivo |
|---|---|---|
| **Full Loader** | O usuário precisa extrair um dataset completo sem mecanismos de rastreamento (sem colunas de data/delta). Sintomas: tabelas pequenas/de referência onde o volume de dados permite fazer *drop-and-insert* contínuo sem perdas graves de performance. | `./Data Engineering Design Patterns/Data Ingestion/full-loader.md` |
| **Incremental Loader** | O processamento do usuário está lento demais pois extrai tudo. Sintomas: a tabela origem possui partições de tempo ou colunas de rastreamento (`ingestion_time`) e os dados não sofrem *hard deletes*, permitindo extrair e anexar (*append*) apenas os deltas mais recentes. | `./Data Engineering Design Patterns/Data Ingestion/incremental-loader.md` |
| **Change Data Capture (CDC)** | O usuário clama por processamento em tempo real (baixa latência) ou necessita capturar exclusões físicas (*hard deletes*). Sintomas: gargalos de lote (batch) sendo contornados por fluxos contínuos ouvindo diretamente o *log de commit* do BD (ex: Debezium). | `./Data Engineering Design Patterns/Data Ingestion/change-data-capture.md` |
| **Passthrough Replicator** | O usuário precisa copiar dados integralmente (1:1) entre ambientes (ex: de produção para homologação) sem alterar estruturas lógicas, mantendo metadados brutos impecáveis para *debugging* ou reprodução de falhas. | `./Data Engineering Design Patterns/Data Ingestion/passthrough-replicator.md` |
| **Transformation Replicator** | O usuário precisa espelhar dados de produção, mas há alto risco de conformidade com informações pessoais (PII). Sintomas: a replicação exige ofuscamento transacional explícito ou supressão de colunas confidenciais (anonimização) no meio do trajeto. | `./Data Engineering Design Patterns/Data Ingestion/transformation-replicator.md` |
| **Compactor** | O usuário relata *metadata overhead* ou lentidão absurda para ler o *data lake*. Sintomas: uma proliferação incontrolável de micro-arquivos fragmentados (*small files*) prejudicando severamente operações de I/O de leitura, exigindo fusão transacional (*optimize*/*vacuum*). | `./Data Engineering Design Patterns/Data Ingestion/compactor.md` |
| **Readiness Marker** | Consumidores acusam leitura de diretórios vazios ou incompletos. Sintomas: ausência de orquestração unificada, forçando o injetor a afixar um arquivo-flag (`_SUCCESS`) que chancelará fisicamente a completude para que agentes consumidores baseados em *polling* iniciem. | `./Data Engineering Design Patterns/Data Ingestion/readiness-marker.md` |
| **External Trigger** | Os jobs do usuário desperdiçam rios de dinheiro monitorando e rodando checagens em pastas que raramente recebem dados (*pull overhead*). Sintomas: é vital inverter o fluxo passivo, ativando *pipelines* remotamente via webhooks/API apenas quando o evento efetivamente ocorrer (*push*). | `./Data Engineering Design Patterns/Data Ingestion/external-trigger.md` |

## Categorias de Domínio

Para facilitar sua cognição arquitetural, os padrões atualmente mapeados dividem-se logicamente nas seguintes classificações em Ingestão de Dados:

- **Mover e Popular (Criacionais):** *Full Loader* e *Incremental Loader* — Operações basilares de movimentação originária e massiva de dados que fundam o nascimento dos conjuntos analíticos.
- **Ajustar e Otimizar Estruturas (Estruturais):** *Passthrough Replicator*, *Transformation Replicator* e *Compactor* — Operam sobre a estrutura intrínseca, sanitização legal (PII) e otimização física do estado do armazenamento (storage footprint).
- **Garantir Estado e Reatividade (Comportamentais):** *Change Data Capture*, *Readiness Marker* e *External Trigger* — Governam a reatividade sistêmica, acionando engrenagens de execução ou atestando barreiras de estado imutável para a orquestração distribuída.
---
