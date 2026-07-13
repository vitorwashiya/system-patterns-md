# Concurrent Runner

**Categoria:** Fluxo de Dados

## 🎯 Objetivo
Desobstruir a infraestrutura paralela e orquestrar e colmatar com liberdade fluxos múltiplos não interligados de datasets da arquitetura orquestrada de forma livre em concorrência libertando restrições no tempo no scheduler.

## O Problema
Apesar de atuar com sucesso com Single Runners, o engarrafamento das pipelines em "Backfilling" para tabelas imensas das rotinas da equipa que são independentes nos dados gera atrasos absurdos aos consumidores a jusante. Como não existem dependências ou "states" a manter nas tabelas alvo orquestradas e todas derivam estritamente independentes em cada via do dia da carga pretendemos abolir limites e atuar rápido e solto.

## A Solução
Como isentas dependências nativas desliga perfeitamente restrições operativas na UI do motor ativando o "Concurrent Runner". Requer ativamente remover ou alargar para limites altos (`max_active_runs=5`) as fronteiras na topologia macro permitindo os agendadores pegarem instâncias disponíveis puramente livres de barreiras nas invocações base em fila de disparo desde que tenham "slots livres" de cálculo puro no servidor local configurados para agilizar instantaneamente processamentos orquestradores nas filas paralelas de execução nas orquestrações lógicas das partições de dados independentes.

## Prós e Contras
- **Prós:** 
  - Excelente e ideal via primária nativa para solucionar com estonteante fluidez falhas profundas do passado repondo partições massivamente (Backfilling puro) ao mesmo instante não bloqueando a agenda das horas subjacentes operacionais na grelha.
  - Atenua grandiosamente latências estritas do sistema eliminando esperas do motor.
- **Contras:** 
  - Tende na desregulação perante Multitenant Environments (espaços partilhados das orquestrações multi-equipa) a exaurir perigosamente e roubar (Resource Starvation) na plenitude pura todos os compute pools ou slots originando congelamentos nas pipelines das vias terceiras operativas.
  - Altamente suscetível na introdução de erros silenciosos estritos lógicos ou fatais se aplicados perante bases de lógica stateful ocultas e não perfeitamente diagnosticadas resultando em partições esquecidas originais ou múltiplas reescritas redundantes das latências orquestradoras assíncronas do estado.