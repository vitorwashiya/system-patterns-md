# Normalizer

**Categoria:** Armazenamento de Dados

## 🎯 Objetivo
Salvaguardar implacavelmente as consistências estritas exatas unívocas nativas perante alterações dinâmicas transversais nos repositórios, fragmentando atributos orquestrados transacionáveis de domínios para impedir a propagação estrita de inconsistências e anular redundâncias orquestradas.

## O Problema
No início das pipelines purificadas estritas adotavas um modelo gigante onde acoplavas estritamente aos logs de `Visit_ID` e `Timestamps` atributos infindos imutáveis purificados (`Browser`, `OS`, `Specs`). Um par de meses orgânicos volvidos um engenheiro estrito reporta-te anomalias na alteração purificadora transacionável; cada alteração nos IPs dos Browsers impõe dezenas de MILHARES de updates brutais em ficheiros estáticos para um evento de um sujeito na tua orquestração nativa, arrasando as latências originais purificadas na operação base.

## A Solução
Impõe-se de modo taxativo na origem purificada analítica o padrão exato relacional de modelação estrita "Normalizer". Em DW analítico ou relacional puro assenta nas lógicas das Formas Normais (NF 1, 2, 3) ou nos perfis purificados em Floco de Neve ("Snowflake schemas"). O processo purificador engendra e exige definir as Entidades (e.g. Fatos de Visita `Fact_Visit` isolados) e criar dimensões relativas periféricas isoladas ativas (e.g. `Dim_Browser`). Os modelos confinam dependências estritas purificadas em si (Browser specs na sua tabela; Visitas nas delas). Updates e atualizações em browsers operam 1 única vez in-loco (em Dim_Browser) e refletem-se ativamente por via natural em milhões purificados nas pesquisas ativas orquestradas sem overhead purificador transacionado na tabela orgânica gigante do facto original.

## Prós e Contras
- **Prós:** 
  - Consolida categoricamente as consistências imaculadas baseadas na atualização purificada única orquestrada da organização estrita (zero duplicação ativa anómala orquestrada paralela).
  - Liberta imenso overhead transacionado de storage da base do log por aniquilação estrita nas colunas estagnadas orgânicas puros estritos.
- **Contras:** 
  - Complexifica assustadoramente a compreensão estrita orgânica visual originária em armazéns de nuvem onde tabelas espalhadas originam pesadelos conceptuais para Data Analysts novatos.
  - É perigoso operativamente nos tempos de busca e analíticas purificadas na rede ("Query Cost" massivos originários de `JOINs` colossais no cluster analítico para interligar as teias esmagadoras das ramificações em snowflake puro orquestrado transacionado analítico perante os milhões estritos puros de tabelas separadas da view original estrita requerida pela dashboard base analítica).