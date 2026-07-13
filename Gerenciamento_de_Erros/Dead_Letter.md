# Dead-Letter

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Evitar a falha de toda uma pipeline de processamento devido à existência de registos inválidos, processando ativamente os dados com boa qualidade enquanto se captura as falhas e as direciona para uma camada morta (dead-letter storage) para posterior análise.

## O Problema
Num cenário de ingestão em streaming (ou mesmo em batch contínuo), podes subitamente deparar-te com dados que não possuem campos vitais, contêm tipos inválidos ou falham em certas funções. Uma abordagem natural (fail-fast) terminaria o trabalho todo. Esta falha seria inaceitável pois interromperia também o processamento de muitos registos bons por causa de um único ou poucos registos ruins (poison pill messages).

## A Solução
O padrão Dead-Letter consiste em encapsular as transformações críticas em blocos `try-catch` (ou identificar falhas com `if-else` e funções "error-safe"). Sempre que uma operação deteta um registo inválido, não interrompe a linha; em vez disso, anota a causa e direcciona o registo para outro caminho (um "side output" do Flink ou um caminho de salvamento alternativo no Spark/Delta). Daqui, poderão ser inspecionados ou possivelmente alimentados a uma "replay pipeline" secundária após resolução na origem.

## Prós e Contras
- **Prós:** 
  - Melhora bastante o tempo de atividade dos sistemas, não penalizando os bons dados; o trabalho prossegue para outras linhas.
  - Oferece grande flexibilidade aos analistas ou equipa de operações que podem avaliar os erros armazenados separadamente.
- **Contras:** 
  - Obriga muitas vezes a uma escrita e manutenção mais exaustiva. Se o erro for generalizado, o sistema não avisa de imediato (esconde falhas fatais).
  - Pode introduzir complicações como os efeitos bola-de-neve no backfilling caso usemos a "replay pipeline" para integrar estes dados atrasados.