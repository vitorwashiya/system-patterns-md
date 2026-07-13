# Change Data Capture

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Fornecer dados da origem para o destino com uma latência muito mais baixa do que as abordagens em batch tradicionais, garantindo também a captura exata das operações subjacentes nos dados (incluindo deleções físicas).

## O Problema
A cadência do `Incremental Loader` (ex: hora a hora) não é suficientemente rápida para os consumidores, que passam muito tempo à espera. Precisamos de enviar registos transacionais (onde modificações ou remoções acontecem constantemente) para o broker de mensagens em (ou muito próximo de) tempo real.

## A Solução
O padrão Change Data Capture (CDC) ingere continuamente as linhas alteradas e removidas ao ler diretamente o registo interno de commit da base de dados de origem (commit log) e fluí-las para um tópico de transmissão de eventos. Essa abordagem garante a baixa latência e intercetações físicas de atualizações e eliminações.

## Prós e Contras
- **Prós:** 
  - Extremamente baixa latência e rápida disponibilização de dados em movimento (data-in-motion).
  - Captura imediata e infalível de inserções, atualizações e de **deleções rígidas** (hard deletes).
- **Contras:** 
  - Elevada complexidade de setup, pois exige integração a baixo nível com a arquitetura de banco de dados (ex: habilitação do commit log nos servidores).
  - Os dados em movimento diferem semanticamente dos estáticos, obrigando a adaptações e considerações para joins assíncronos.
  - Inclui muitos campos técnicos (metadata do CDC) e payload de estado "antes/depois" que pode exigir forte processamento downstream para ignorar o que é irrelevante.