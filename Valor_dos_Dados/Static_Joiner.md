# Static Joiner

**Categoria:** Valor dos Dados

## 🎯 Objetivo
Cruzar os dados em fluxo ou em lotes incrementais que fluem pelas vias com datasets referenciadores imutáveis, colmatando o vazio de dados e incrementando ativamente a espessura contextual de propriedades originais vazias ou limitadas dos datasets da fonte analítica.

## O Problema
Tens equipas na base a receber milhares de pings simples onde as chaves e valores isolados relatam comportamentos mas evadem contexto (ex. "User ID = 5 e Timestamp clicou na ação B"). Os dados são pouco atrativos. Uma outra equipa armazena passivamente informações macro sobre o registo vital dos utilizadores (dados estáticos). Deves fazer este casório para enriquecer a via analítica com o valor holístico, cruzando referências com o detalhe demográfico subjacente do utilizador.

## A Solução
Como se pressupõe o carácter em descanso de um lado e movente noutro invoca-se o Static Joiner. Envolve criar elos entre as características isoladas, identificando um denominador unificador cruzando ambos. Posteriormente as plataformas e engines efetuam cruzamentos lógicos clássicos tipo junção horizontal relacional (JOINs) por meio direto estrito, recorrendo à semântica passiva. Podes aplicar isto com dimensões de ritmo de alteração extremamente escassos como na "Slowly Changing Dimensions".

## Prós e Contras
- **Prós:** 
  - Enriquece brutalmente a valência para as operações downstream entregando o trabalho sujo pré-processado para o Datawarehouse e os ecossistemas BI visuais.
  - Execução padronizada altamente facilitada com funções intrínsecas e transversais a todas as ferramentas orientadas ao paradigma tabular sem necessitar customizações na semântica.
- **Contras:** 
  - Desafios críticos ligados na exata consistência do processamento originada pelo hiato (late data latency); utilizadores estáticos mal atualizados gerarão outputs envelhecidos prejudiciais.
  - Riscos acrescidos para a coesão associada quando lidamos com recuos (backfill) forçando frequentemente os ambientes temporais retroativos na procura precisa da fotografia passada das variáveis das Slow Change Dimensions.