# Horizontal Partitioner

**Categoria:** Armazenamento de Dados

## 🎯 Objetivo
Agilizar profundamente os volumes de leituras em pesquisas temporais e analíticas particionando blocos imensos da tabela de dados ao meio orgânico dos predicados horizontais isolados (diretórios).

## O Problema
Tens dados imensos orquestrados em batch gerando estatísticas de roll-up sobre os 4 dias transatos. Nos primeiros tempos, as leituras liam velozmente as linhas porque a base era pequena. Contudo, 6 meses depois o volume gerou anomalias. Analisaste o plano de execução e o `WHERE event_time > ...` da tua query gasta horas porque, não possuindo indexação prévia ou ordenação isolada e estando todos os dados agrupados indistintamente em blocos contínuos no S3, o motor é obrigado a fazer um 'Full Table Scan' de todos os meses passados que já não importam nada. 

## A Solução
Como os volumes exigem varrimentos lentos aplica-se ativamente o Horizontal Partitioner. Em termos genéricos pegas nos dados e no momento restrito da gravação organizas a escrita baseando a chave estrita e temporal (`event_time`, `year`, `month`). Na nuvem isto traduz-se fisicamente no armazenamento em caminhos explícitos estritos (e.g. `visits/year=2024/month=10/`). Quando as engines (Spark, Hive, Presto) invocam a leitura subsequente (e.g. `WHERE month=10`), a framework exclui à partida milhares de pastas puras de outros meses abstendo o I/O drástico de leitura.

## Prós e Contras
- **Prós:** 
  - Impulso incalculável da performance de leitura pura e analítica perante recortes cronológicos ou geográficos base.
  - Oferece abstração para reescritas idempotentes fáceis (Overwrite pura na partição).
- **Contras:** 
  - Problema clássico infernal estrito dos "Small Files" ou "Metadata Limits" caso a cardinalidade da chave estrita selecionada for absurdamente grande (particionar pelo UID e não pelo Mês gera esgotamento da Metainformação e paralisação pura da listagem).
  - Skew nas partições estritas acarreta latência extrema nas cargas assíncronas do nó lento ("stragglers") ou exige processos de mitigação morosos ativos.