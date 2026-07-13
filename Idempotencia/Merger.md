# Merger

**Categoria:** Idempotência

## 🎯 Objetivo
Manter coerência global aplicando atualizações granulares entre lotes incrementais recebidos de fornecedores versus um conjunto consolidado histórico residente no destino, evadindo processos dispendiosos que exigem purgas totais.

## O Problema
Tens dados a fluir vindos de ferramentas como "Change Data Capture". Devido à exigência do cliente não podes adotar métodos limpos de "truncate e refazer", pois os novos eventos chegam como deltas em formato incrementado (partes e peças atualizadas sobre uma grande realidade passada inalterada). Sobrescrever o destino a 100% destruiria dados válidos ausentes nesse lote parcial diário.

## A Solução
Como as modificações fornecidas só contemplam subconjuntos mutáveis, é imperativo usar o padrão Merger. Combinando lógicas declarativas de instruções consolidadas num dialecto (ex: o comando estandardizado `MERGE` de ferramentas analíticas/SQL), o padrão compara num só fôlego chaves primárias. Num match (coincidência), aciona um `UPDATE` local ou soft `DELETE`. Numa ausência de correlação (missing) executa o clássico `INSERT`.

## Prós e Contras
- **Prós:** 
  - Permite manter atualizados repositórios persistentes volumosos apenas incidindo operações IO diretas nas fatias que mudam.
  - Aumenta incrivelmente a adaptabilidade em cenários dinâmicos e mutáveis do que uma pipeline apenas suportada em append.
- **Contras:** 
  - Pode tornar-se lento e agressivo em consumo face ao facto de não funcionar na abstração mas a varrer diretamente as bases para cruzamentos comparativos (IO).
  - Problemas persistentes associados ao facto de inviabilizar abordagens coesas se subitamente houver backfills do histórico incrementado não transacionais (pede uma adaptação do utilizador via mecanismos de versionamento extra).