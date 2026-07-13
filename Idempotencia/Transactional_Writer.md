# Transactional Writer

**Categoria:** Idempotência

## 🎯 Objetivo
Exigir entregas íntegras através de esquemas semânticos do "tudo ou nada". Este mecanismo encobre leituras parcelares acidentais dos sistemas em execução bloqueando acessos imaturos dos recetores finais a processos instáveis não terminados por parte da rotina de escritura subjacente.

## O Problema
Para baixar custos com a fatura na cloud os teus batch jobs recorrem a rotinas dinâmicas sobre nós não estáveis e preemptíveis. As pipelines processam registos para base, mas quando a cloud decide deitar o servidor abaixo a meio da rotina e move a thread para outra máquina o trabalho reinicia, lançando caos nos consumidores a juzante porque os registos intermédios entretanto depositados provocaram uma avalanche incontrolável de duplicados imperfeitos sem fim à vista.

## A Solução
Como as leituras incompletas expostas geram perdas reputacionais inaceitáveis nos repositórios recetores, é mandatória a aplicação do Transactional Writer. Seja este de forma estritamente delegada ou controlada imperativamente pelos programadores via as declarações de `BEGIN TRANSACTION ... COMMIT ... ROLLBACK`. Os resultados finais da tua atividade encontram-se selados perante perspetivas alheias no limiar dos teus pacotes atómicos (isolamento), libertando as confirmações globalmente visíveis somente sob termo íntegro e cabal da tarefa.

## Prós e Contras
- **Prós:** 
  - Permite re-processos com re-tentativas impiedosas sem perigos paralelos num limiar em rede; o ecossistema age infalivelmente face aos consumidores evitando surpresas da visibilidade antecipada.
  - Promove a capacidade universal e altamente difundida a todos os modelos clássicos de BD ou Formatos Tabulares que sustentam propriedades puras de semânticas ACID.
- **Contras:** 
  - Requer de modo generalizado passadas demorosas e sobrecargas logísticas severas em instâncias fragmentadas face às necessidades de sincronismo, reduzindo latência visível (sobretudo no Streaming para micro-batch delays).
  - É impossível para arquiteturas restritas na ausência da compatibilidade nativa da framework para o destino selecionado (ex. certas escritas base no Apache Spark para outputs que ignoram suporte isolacional global).