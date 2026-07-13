# Dynamic Late Data Integrator

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Mitigar as vulnerabilidades associadas ao processamento "em vazio" da abordagem estática, implementando mecanismos capazes de acionar reavaliações do passado apenas nas partições em que verdadeiramente se introduziram dados atrasados.

## O Problema
O negócio pretende uma retrospetiva ilimitada nos dados e não só de 15 dias, devido a processos empresariais tardios. Rodar o Static Late Data Integrator a abranger períodos maiores consome quantias exorbitantes de processamento. Pretendemos agora saber "como não reprocessar tudo" e executar o backfill inteligentemente só para quem sofreu injeções de late data.

## A Solução
Requer-se aqui gerir um rastreador em estado, vulgo "State Table", registando a última vez que a partição analítica foi compilada com êxito versus a data da "última atualização efetiva na origem (update time)". A pipeline vai assim comparar se houve mexidas subjacentes ocorridas algures depois do nosso registo final, usando estes deltas e `Dynamic Task Mapping` da orquestração para criar instâncias computacionais apenas direcionadas aos tempos verdadeiramente alterados.

## Prós e Contras
- **Prós:** 
  - Elevada otimização da computação e eficiência em pipelines retroativas profundas ou ilimitadas.
  - Permite a deteção ativa que o Static Integrator descura a bem de um período cego de segurança.
- **Contras:** 
  - Bastante mais suscetível a erros de sobreposição e paralelismo não desejado (race-conditions com várias pipelines a reavaliarem subitamente as mesmas instâncias).
  - Aumenta expressivamente a complexidade global em engenharia, ao obrigar quer o fornecedor quer o repositório a exporem com exatidão marcas ativas de modificação (como a view INFORMATION_SCHEMA) que variam entre nuvens.