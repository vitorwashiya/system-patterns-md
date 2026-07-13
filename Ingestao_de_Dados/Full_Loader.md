# Full Loader

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
O objetivo deste padrão é ingerir um conjunto de dados completo de cada vez. É particularmente útil para inicializações de bases de dados (bootstrapping) ou para a geração de conjuntos de dados de referência (reference datasets).

## O Problema
Por vezes, precisamos de ingerir dados de fornecedores externos que alteram pouco ao longo do tempo. O problema surge quando este fornecedor não define nenhum atributo (como uma data de atualização) que nos permita identificar apenas os registos que foram alterados desde a última ingestão.

## A Solução
A solução passa por utilizar o padrão Full Loader, que implementa uma operação de extração e carregamento (Extract and Load - EL) ou de extração, transformação e carregamento (ETL). Basicamente, lê-se o conjunto completo de dados da origem e substitui-se o conjunto existente no destino de forma a ter sempre os dados mais recentes.

## Prós e Contras
- **Prós:** 
  - Simplicidade de implementação, requerendo frequentemente apenas scripts nativos ou simples ferramentas de cópia.
  - Com volumes de dados de crescimento lento, a infraestrutura terá necessidades constantes e funcionará de forma estável.
- **Contras:** 
  - Se o volume de dados duplicar repentinamente, os tempos de execução aumentarão, e os limites estáticos de hardware poderão causar falhas.
  - Riscos de consistência durante a operação de reescrita total, que podem resultar na exposição de dados parciais aos utilizadores caso falhe a meio, exigindo gestão via transações ou visualizações abstratas (views).