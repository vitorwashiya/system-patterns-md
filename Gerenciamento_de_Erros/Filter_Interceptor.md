# Filter Interceptor

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Observar criticamente e de modo preciso a agressividade ou efetividade de condições limitadoras num fluxo (os filtros lógicos de seleção), captando informação analítica sobre o número exato de registos descartados num ponto antes da sua omissão total.

## O Problema
Numa operação normal em batch, libertaste uma atualização ao código e, repentinamente, verificas com sobressalto que em vez de 15% dos teus dados rejeitados ou não aptos, tens agora quase 90% dos mesmos filtrados. Tens a tarefa urgente de discernir sem demora se a origem é um problema orgânico e severo nos dados entrados ou alguma regressão estúpida codificada de teu lado.

## A Solução
Como as engrenagens de processamento ocultam amiúde em "físico" o plano combinatório para acelerar percursos (otimização de predicados), torna-se vital usar o padrão Filter Interceptor. Envolve não só declarar a cláusula ou função impeditiva como intercetar essa execução e registar via "acumuladores" em APIs nativas ou colunas estendidas nalgum CTE intermédio (`subquery`) exatamente quantas linhas bateram na recusa para um determinado motivo isolado.

## Prós e Contras
- **Prós:** 
  - Auxílio imenso na identificação remota da pureza do tráfego ou monitoramento cirúrgico de qual parte da restrição ativada quebra na verdade o modelo.
  - Fundamental para a validação pragmática de pipelines, pois permite entender exatamente que tipo de falhas humanas ou erros sistémicos geram dados não observáveis na saída.
- **Contras:** 
  - Impacta ligeiramente no tempo das queries se as lógicas exigirem agregações e recolhas por parte da API paralela ou se induzirem subqueries intermédias em SQL onde se poderiam resolver nativamente via `WHERE`.
  - Requer adaptações complexas de janelas e estados no streaming para correlacionar corretamente estatísticas interceptadas num quadro temporal estrito.