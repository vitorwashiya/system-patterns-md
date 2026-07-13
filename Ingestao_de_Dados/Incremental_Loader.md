# Incremental Loader

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Este padrão tem como foco carregar dados de forma incremental, processando apenas novas partes de um conjunto de dados. Isso reduz drasticamente a necessidade de hardware em comparação com cargas completas diárias.

## O Problema
Tens processos que recebem dados que aumentam de volume contínua e aceleradamente, por exemplo de sistemas legacy. Carregar todo o conjunto de dados a cada execução deita abaixo a infraestrutura de dados ou é simplesmente demasiado demorado e dispendioso, exigindo que se leiam apenas os registos adicionados desde a última carga.

## A Solução
A solução divide-se em duas abordagens:
1. Uso de uma **coluna delta** (como data de ingestão) para saber os novos registos a carregar.
2. Uso de **partições de tempo**, onde o trabalho de ingestão é executado sobre partições discretas de tempo do armazenamento que já contêm apenas os dados a ingerir.

## Prós e Contras
- **Prós:** 
  - Volume de ingestão muito mais reduzido face à abordagem de Full Load.
  - Execuções mais rápidas e otimizadas em tempo e custo.
- **Contras:** 
  - Problemas na gestão de **hard deletes** (registos removidos na origem não o são automaticamente no destino se não houver um soft delete explícito).
  - Um reprocessamento massivo (backfilling) de longo prazo reverte a lógica quase para um Full Load se não for cuidadosamente configurado.
  - Exige a persistência e manutenção do estado temporal da última ingestão para a variante de coluna delta.