# Windowed Deduplicator

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Assegurar uma semântica de processamento "exatamente uma vez" (exactly-once) para registos de dados distintos numa dada arquitetura, evitando as consequências prejudiciais do processamento de mensagens duplicadas.

## O Problema
Tens uma camada de stream a gravar eventos. Graças a processos de segurança como os do Dead-Letter ou as normais tentativas automáticas de retoma do sistema (retries) associadas a problemas transitórios (rede/latência), o sistema que envia dados emite múltiplos exemplares idênticos do mesmo registo de evento, pondo em risco a contagem final ou os consumos nos recetores baseados nestes volumes.

## A Solução
No processamento batch, o Windowed Deduplicator confia simplesmente numa função que examina todo o conjunto atual e escolhe apenas o primeiro (ex: via `dropDuplicates` ou uma instrução SQL como `WINDOW` com `ROW_NUMBER()`). Em streaming, este padrão define janelas de tempo, armazenando os valores num state store que gere um estado transacional, verificando se cada evento que está a entrar já passou pela pipeline dentro daquela janela temporal e o ignorando se for caso disso.

## Prós e Contras
- **Prós:** 
  - Facilita relatórios imaculados onde o "ruído" originado pelo ambiente de engenharia não atinge a superfície analítica.
  - Evita dependências pesadas face a consumidores, isolando este problema na ingestão pura.
- **Contras:** 
  - Custo em "espaço versus tempo": As instâncias long-running em streaming consomem muito espaço ao memorizar os IDs, exigindo janelas limitadas de cache.
  - Deduzir dados não significa necessariamente que não falte nada ou que as entregas serão idênticas numa nova repetição global.