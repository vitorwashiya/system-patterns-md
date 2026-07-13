# Stateful Merger

**Categoria:** Idempotência

## 🎯 Objetivo
Enriquecer a fusão de incrementos com capacidades avançadas baseadas num estado contínuo (State Table), com o foco supremo na retenção de uma capacidade limpa e idêntica de reverter operações falhadas a momentos históricos coerentes no processo de backfilling de pipelines incrementais.

## O Problema
Os teus processos usam o padrão Merger para aplicar sincronizações a repositórios em Delta Lake. Quando ocorrem anomalias sérias, a equipa pede intervenção ou correção. Contudo, reler um passado incrementado resulta em lacunas dolorosas: o "merge" normal que lê pacotes desatualizados destrói atualizações mais maduras presentes no armazenamento final que não aparecem nesse extrato passado. Não existe coerência numa reversão manual sem apagar ou poluir.

## A Solução
Para garantir a capacidade autêntica de reversão nas janelas e estancar as incongruências do lote em pipeline retroativa, usas o padrão Stateful Merger. Incluis uma tabela suplementar estrita que assenta nas versões exatas criadas antes da operação de `MERGE` processada e associa estas versões às horas do processo orquestrador. Se a orquestração forçando uma retoma do passo passado vir a diferença desta State Table, executa uma ação intermédia protetora de RESTORE explícito para a realidade temporal original das tabelas pré-fusão.

## Prós e Contras
- **Prós:** 
  - Blinda operações em lote assíncronas nas falhas severas de negócio onde restaurar instantes é imperativo.
  - Reduz drasticamente as incoerências por omissão associadas a ficheiros fragmentados nos updates não idempotentes por natureza nas fusões tradicionais.
- **Contras:** 
  - Confiança mandatória exclusiva a ecossistemas suportados com rastreio granular transacional (ex: motores compatíveis com "time travel").
  - O overhead extra de ter de manter lógicas assentes numa ou mais tabelas espelho de estado, aumentando o fardo e as rotinas de retenção na gestão do sistema.