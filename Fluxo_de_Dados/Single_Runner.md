# Single Runner

**Categoria:** Fluxo de Dados

## 🎯 Objetivo
Bloquear rigorosamente concorrências de fluxos inteiros ao restringir e encapsular as instâncias e pipelines globais orquestradoras numa perspetiva puramente singela e unitária em ação isolada.

## O Problema
Validaste orquestrações perante a lógica do teu estado contínuo (Stateful logic/Incremental sessionizers), que funciona bem num ciclo de testes e invocações de gatilho manual interativas. Ao passar à fase mecânica automatizada da pipeline orquestradora em schedule a tua base requer nativamente de evitar qualquer cruzamento estrito assíncrono perante os estados, mas ainda não definiste a lógica transacional no controlador.

## A Solução
Nativamente os trabalhos com dependências sucessivas obrigam à linearidade perante toda a infraestrutura macro orquestradora. Instancia-se e bloqueia de modo explícito a paralelização na orquestração pelo padrão "Single Runner", declarando com assertividade `max_active_runs=1` ou limites puramente estritos no Data Factory em fila fechada nativamente. Garante-se assim em pleno controlo absoluto que instâncias pendentes das pipelines a aguardar a agenda nunca forçarão simultaneidades em bases restritivas assentes nativamente nos "previous status" nas lógicas stateful purificadas e geridas pelo estado encadeado subjacente global.

## Prós e Contras
- **Prós:** 
  - Garante e purifica na abstração lógica da arquitetura as injeções orgânicas incrementais assentes da lógica temporal subjacente na partição (não requer lógicas complexas puras intrincadas para travar estados e race conditions de commits originais do código de bases).
  - Configuração banal perfeitamente integrada "Out of the box" das principais plataformas macro sem necessitar sequer repensar ou reinventar as dinâmicas transativas locais operacionais.
- **Contras:** 
  - Terrivelmente nefasto no desempenho perante ações retrospetivas em lotes gigantes puramente independentes (Backfilling); encadeando morosamente partições estritas que seriam mais eficientes massivamente calculadas simultâneas orquestradas num paralelismo alargado.
  - Adiciona pressões fatais no SLA purificado no sistema e propaga inevitavelmente engarrafamentos analíticos estritos perante atrasos ou "stragglers" (se demora ou encalha hoje 2h arrasta ativamente em cadeia todas as instâncias em delay infinito amanhã nas latências restritivas base).