# Vertical Partitioner (Storage)

**Categoria:** Armazenamento de Dados

## 🎯 Objetivo
Isolar na organização do armazenamento puro as propriedades inorgânicas ou imutáveis num repositório estrito, prevenindo a duplicação colossal de atributos densos mas puramente repetidos ao longo de uma mesma entidade.

## O Problema
Geres pipelines com propriedades extremamente mutáveis das sessões (visitas diárias de sites) mas atrelas no mesmo log os dados colossais e pesados estritamente estáticos referentes ao modelo detalhado do Hardware dos Utilizadores, versões, OS, etc. O teu I/O de escrita atual duplica sem regra atributos enormes imutáveis redundantes por cada linha do event stream orquestrado, entupindo fortemente espaços na Cloud em armazenamento analítico.

## A Solução
Usas logicamente a abstração do Vertical Partitioner focado ao armazenamento. Quebras estritamente o layout do teu dado numa base de separações. Abstrais por junções os dados que sofrem mutação agressiva e os que estão inalterados (os estáticos na base e PIIs) e separas as secções enviando colunas estritas via SQL Subqueries nativas ou Mappers nativos para Repositórios em separado estritos ("Dicionários de Referências estáticos puros" num local e "Visitas ativas com os chaves e pontes ID" noutro).

## Prós e Contras
- **Prós:** 
  - Atenua grandiosamente o custo do "Data Overhead" ao aniquilar estritamente repetições de atributos exaustivos na persistência final (Redução substancial do storage footprint).
  - Extremamente benéfico face ao império das regras legais (GDPR), onde os utilizadores têm os PIIs apagados via `DELETE` purificado estritamente localizado (ver Padrões de Segurança).
- **Contras:** 
  - Complexifica grandiosamente as rotinas nativas de queries originais que são forçadas a transacionar `JOINs` brutais originários (Data split) entre partições dispares no modelo físico no momento exato do acesso.
  - É perigoso na coerência estrita de sistemas no "Polyglot persistence" (bases de dados cruzadas híbridas estritas).