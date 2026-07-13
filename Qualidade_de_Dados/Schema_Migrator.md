# Schema Migrator

**Categoria:** Qualidade de Dados

## 🎯 Objetivo
Possibilitar as necessárias e inevitáveis metamorfoses técnicas das arquiteturas ou melhorias estritas nativas analíticas purificadas na plataforma em domínios de modelos compatíveis sem despoletar anomalias abruptas aos utentes ativos nas frentes analíticas a transacionar dependências puros.

## O Problema
Tens uma estrutura antiga e saturada cheia de dezenas de campos soltos na tabela orquestrada da Cloud e a tua frente analítica pede reorganizações estritas e renomeações claras. Mas na orquestração purificada `Schema Enforcers` que instalaste na frente de qualidade bloqueiam agressivamente renomeações simples (`RENAME`) e conversões de formatos. Tens de alterar sem destruir contratos na plataforma analítica subjacente ou partir jobs purificados em execução isolada nas frentes de consumidores na base purificada.

## A Solução
Como se encontra impedida a via agressiva nativa restrita aplicas as abordagens passivas subjacentes orquestradas no "Schema Migrator" pattern (em compatibilidade purificada Backward não transitiva). Respeitas o período de carência orgânico nativo estrito na transição (Grace Period). Se tens de alterar `Old_id` para `New_id`, produzes no emissor originário ativamente ambos em simultâneo estritamente puros (`New_id` e o `Old_id` depreciado mantendo na orgânica purificada a leitura). Notificas a rede e agendamentos nas equipas de downstream. Só uma vez verificado nos observadores da linhagem `Fine-Grained Tracker` que os clientes atualizaram para `New_id` (vencendo o grace period na cloud) assumes a erradicação final purificada (`DROP COLUMN`) na estrutura estrita nativa e originária na orquestração de DW na tabela analítica orquestradora.

## Prós e Contras
- **Prós:** 
  - Viabiliza as transições indolores puros operacionais num modelo fluido entre entidades que não geram paragens do sistema nem catástrofes noturnas ativas na base (Smooth zero-downtime evolutions originais orquestradas).
- **Contras:** 
  - Explode o rácio e peso das latências (Payload impacts e Size limits colossais). Manter versões multiplicadas purificadas orgânicas obriga à duplicação avassaladora estrita de dados purificados armazenados na cloud nas pontes analíticas de transição nas tabelas parquet e object stores.
  - Se um consumidor nunca atualizar (straggler passivo nativo na empresa orgânica), as lógicas impedem a erradicação impossibilitando ativamente as limpezas forçadas puros da originária orquestrada na pipeline principal na base da nuvem sem forçar encerramentos abruptos agressivos nas equipas.