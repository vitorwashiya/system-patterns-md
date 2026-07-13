# Late Data Detector

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Identificar de imediato eventos atrasados numa infraestrutura, nomeadamente registos capturados ou lidos depois da conclusão processual natural desse mesmo contexto de dados ou janela temporal (partitions ou states).

## O Problema
Eventos como visitas de utilizadores deveriam fluir para a tua plataforma com até 15 segundos de atraso. No entanto, se um cliente entra num túnel sem internet, os pacotes aguardam até o sinal voltar, resultando em dados injetados mais tarde (minutos ou horas depois) cujos carimbos de ocorrência já não coincidem com o tempo normal. O teu sistema não sabe de antemão lidar com este hiato e os processamentos perdem sincronização.

## A Solução
Esta é a base estrutural focada em "detetar". O Late Data Detector baseia-se na criação da chamada "Watermark" (marca d'água temporal). Ele guarda e analisa o tempo de evento máximo observado (através das medições do processamento ou dos IDs de partição) e subtrai uma tolerância autorizada. Registos de origem que venham com tempos mais tardios do que esta Watermark são formalmente rotulados como "Atrasados", quebrando assim o dogma de se processarem automaticamente. 

## Prós e Contras
- **Prós:** 
  - Cria mecanismos extremamente saudáveis para o estado interno de processos em streaming (ex: fecho de janelas).
  - É incorporado nativamente e abstraído via software de processamento modernos (como o Flink e Spark Structured Streaming).
- **Contras:** 
  - Requer estratégia de tratamento do atraso. Algumas engines lidam simplesmente apagando registos o que acarreta perdas reais de dados.
  - Algumas abordagens de agregação de tempos falham face a distribuições desiguais de chegada de dados em diferentes tópicos ou partições concorrentes.