# Fast Metadata Cleaner

**Categoria:** Idempotência

## 🎯 Objetivo
Limpar ou remover de forma eficiente conjuntos de dados anteriores operando exclusivamente a nível lógico (metadados), eliminando redundâncias num fluxo para atingir idempotência rápida na regeneração.

## O Problema
Tens uma pipeline que reescreve diariamente volumes massivos de terabytes. Atualmente usas comandos como `DELETE FROM` baseados em filtros temporais e a performance está péssima pois o motor gasta imensos ciclos nas varreduras. Queres reter o comportamento limpo da sobrescrita (idempotência) para os casos de backfilling ou falha, mas evitar a latência da exclusão a nível físico.

## A Solução
Como as operações de metadados são frequentemente mais rápidas que a iteração em massa em dados, o padrão Fast Metadata Cleaner ataca o repositório utilizando comandos da base de dados como `TRUNCATE TABLE` ou `DROP TABLE`. Ele decompõe a organização para uma granularidade que sirva estas declarações de apagamento a nível da tabela macro em substituição das eliminações cirúrgicas de linhas.

## Prós e Contras
- **Prós:** 
  - Limpezas massivas em frações de segundos.
  - Otimiza severamente abordagens de reescrita pura, promovendo idempotência direta na orquestração.
- **Contras:** 
  - A granularidade da tua exclusão impõe fronteiras duras para re-execuções. Se o backfill tiver de ser num só dia, mas a partição truncável mais pequena for de uma semana, terás de reprocessar obrigatoriamente 7 dias.
  - Implica lidar com os limites rígidos dos provedores Cloud para o número de partições ou de tabelas.