# Checkpointer

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Evitar a perda de referências e evitar o reprocessamento longo de todo um conjunto histórico contínuo (em cenários que envolvem leituras contínuas), gerando uma proteção formal contra falhas de continuidade processual garantindo marcos temporais intermédios salvaguardados.

## O Problema
Tens operações contínuas nos eventos a passar em streaming que realizam contagens. Num processo de dezenas de dias a laborar ininterruptamente, ficas apreensivo pois se houver uma falha súbita catastrófica (fatal error) nos recetores ou no sistema (falha elétrica num data node) perdes não apenas toda a janela de processamento atual, mas serás instado a reler todo o repositório em fluxo desde a origem dos tempos pois não há "partições lógicas" definidas a ajudar no restabelecimento.

## A Solução
Em vez da falha-cega do processo de read-and-hope, impõe-se implementar Checkpointing. A pipeline salva intencionalmente, sob configuração nativa dos frameworks (como no Apache Spark e Apache Flink) ou via acionamento expresso (`commit()` num cliente de Kafka), o seu "offset de leitura e progresso de estado" para armazenamento remoto durável e infalível, frequentemente object stores seguros.

## Prós e Contras
- **Prós:** 
  - Protege radicalmente a execução em long-running de perdas incalculáveis de retrocesso de dados calculados e tempo real decorrido de computação.
  - Integrado de forma formidável por serviços e bibliotecas estandardizadas minimizando a introdução manual de marcadores de limite.
- **Contras:** 
  - A complexidade no custo e na latência em tempo real; fazer commits intensivos prejudica a performance linear ao pausar a receção com micro demoras de confirmação de registo a sistemas persistentes.
  - Oferece uma falsa promessa do paraíso de "entrega exata". Porque se uma partição falha na iminência imediata mas prévia ao seu commit, os dados já enviados resultarão numa duplicação aquando da sua retoma da posição garantida mais recente (At-least-once processing).