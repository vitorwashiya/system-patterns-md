# Readiness Marker

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Sinalizar à cadeia completa de consumidores de dados o instante mais apropriado de consumir a base. Garante-se desta forma que todos os processamentos que se baseiem nestes ficheiros acedam aos dados na sua forma completada.

## O Problema
Várias equipas executam modelos complexos em cima dos ficheiros diários criados por ti na zona de exposição do Data Lake. Devido ao desfasamento horário, os consumidores ativam com regularidade pipelines enquanto os dados diários continuam em transformação na origem. Como eles encontram apenas um conjunto de dados processado a 50%, geram falhas, visões inconsistentes em relatórios, perdas analíticas e desconfiança contínua face aos dados.

## A Solução
No padrão "Readiness Marker", em vez dos componentes acionarem pipelines no tempo de processamento à sorte (em cron-jobs predeterminados), o gerador assume como missão emitir de forma passiva um aviso ou "sinalizador explícito" da sua terminação. Isto engloba usar marcadores estáticos vazios do tipo "SUCCESS", criar uma partição extra temporária que denote terminação formal de lote, ou registar o sucesso formal em ferramentas de orquestração do ecosistema.

## Prós e Contras
- **Prós:** 
  - Abordagem simples, altamente confiável e não-intrusiva.
  - Elimina de imediato quebras ou corrupções em relatórios da componente de exploração que derivam da interceção de escritas simultâneas.
- **Contras:** 
  - Falta de mecanismos automáticos para aplicar ou obrigar os utilizadores a consultar as verificações estáticas (apenas acordos informais impedem violações antecipadas).
  - Perigo ao usar partições baseadas na hora exata com dados super atrasados, gerando problemas ao reabrir e processar dados que a equipa já assumiu estáticos e terminados.