---
# Readiness Marker

## Classificação
- **Tipo:** Comportamental
- **Escopo:** Classe

## Intenção
Afixar indicadores físicos convencionais de flag de encerramento delimitando publicamente a completude inquestionável das gravações finais em domínios assíncronos não transacionais de object storage, neutralizando leituras de frações imperfeitas.

## Também Conhecido Como
Flag File, Success File.

## Motivação (Contexto)
Organizações isoladas consumindo datasets vitais criados esporadicamente lidam frequentemente com incidentes críticos de leituras silenciosas parcialmente defeituosas em pastas do Lakehouse em processamento. Sem gatilhos diretos centralizados as fatias orquestrais downstream necessitam inspecionar passivamente um aval imutável emitido nativamente pelo processo injetor original antes de agir.

## Aplicabilidade
- Use o Readiness Marker quando:
  - Assimetrias radicais entre ecossistemas inviabilizam notificações push robustas por parte da criação originária.
  - Plataformas de armazenamento baseiam-se em lógicas fragmentadas (diretórios isolados em Cloud Storages) desprovidas de commits ACID nativos globais em diretórios massivos.
  - Leitores necessitam sondar ciclicamente sub-partições de lote estáticas no modelo Pull sem intervenção humana direta.

## Estrutura e Participantes
- **Ingestion Job:** Entidade despachante primária escrevendo ativamente fluxos na malha particionada final.
- **Marker / Flag File:** Objeto final de certificação, de conteúdo arbitrário com nomenclaturas fixadas universalmente (ex: `_SUCCESS`, `_COMPLETED`) injetado após fechamento bem-sucedido restritivo.
- **Consumer Sensor:** Mecanismo leitor remoto dependente efetuando checagens pull iterativas ciclicamente sobre uma URI exata pausando invocações ativas à jusante até positivar o veredicto da infra.

## Colaborações
No instante derradeiro, sucedendo todo o fluxo pesado e o esgotamento dos retornos, o Ingestion Job aloca e consolida pontualmente o arquivo Flag delimitador na ponta. O Consumer Sensor bloqueia submissões à base principal indefinidamente sob latência de checagem interrogativa até rastrear magicamente o surgimento desse aval.

## Consequências
- **Prós:**
  - Evidencia isolamento soberbo em arquiteturas imensas e complexamente desconectadas de provedores variados sem implementações adicionais elaboradas.
  - Altíssima adesão natural das abstrações de frameworks open source em Spark que assinalam convencionalmente esse comportamento subjacente out-of-the-box.
- **Contras:**
  - Consumo vampiroso indesejado atrelado à natureza do polling iterativo persistente estagnando em slots ociosos e recursos limitados de schedulers orquestradores nativos.
  - Ausência mandatória restritiva: sem uma barreira efetiva estrutural de permissão física, agentes mal operados ignorando o verificador de flag vão inadvertidamente violar partições ativas.

## Implementação (Exemplo de Código)
```python
# Airflow Sensor iterativo configurado explicitamente no padrão pull nativo de espera passiva
from airflow.sensors.filesystem import FileSensor

next_partition_sensor = FileSensor(
    task_id='wait_for_partition_success_marker',
    filepath=f'{input_data_file_path}/_SUCCESS',
    # Modo reschedule evita manter o worker inútil paralisado iterando na máquina por horas ativamente
    mode='reschedule'
)
```

## Padrões Relacionados

* **External Trigger:** A contramedida estrutural avançada oponente de viés "Push-Based", implantando emissões imediatas para destituir gargalos e longos silêncios inúteis na espera flag-checking de eventos esporádicos dinâmicos.

---
