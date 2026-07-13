# Passthrough Replicator

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Este padrão foca-se na replicação de um conjunto de dados do ambiente de origem para outro local, copiando exatamente a mesma informação num cenário em que não é possível (ou não faz sentido) aplicar lógica adicional.

## O Problema
Ao replicar ambientes de desenvolvimento e pré-produção baseados em dados de produção, encontras uma API ou processo gerador que **não é idempotente**, ou seja, chamá-la de novo gera resultados diferentes. Como precisas que os dados sejam idênticos em todos os ambientes para testes, necessitas de uma abordagem que copie os dados fisicamente do ambiente produtivo de forma exata.

## A Solução
O Passthrough Replicator usa ferramentas simplificadas a nível computacional (como o comando `COPY` do armazenamento) ou ao nível da infraestrutura para copiar diretamente ficheiros ou dados do ambiente principal para os outros ambientes. Ele elimina na íntegra a camada de transformação para preservar perfeitamente o estado exato dos dados sem interrupções e conversões de formatos.

## Prós e Contras
- **Prós:** 
  - Grande simplicidade e fiabilidade; tem muito pouco código para falhar.
  - Rápida sincronização nativa entre ambientes sem correr risco de desconfigurar dados com reinterpretações silenciosas de sistema.
- **Contras:** 
  - O facto de ligar ambientes pode levantar riscos de quebras de segurança; os processos do ambiente produtivo podem ficar congestionados a servir os restantes ambientes (latência).
  - Risco sério de copiar dados pessoalmente identificáveis (PII) sensíveis da produção sem mascaramento ou adequação.