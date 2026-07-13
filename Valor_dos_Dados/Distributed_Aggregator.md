# Distributed Aggregator

**Categoria:** Valor dos Dados

## 🎯 Objetivo
Viabilizar transformações macro orientadas em grande escala de métricas compostas analiticamente sem sofrer dos impasses ou bloqueios sistémicos associados aos rebentamentos causados pela limitação isolada de um nó físico na análise de volumes estúpidos de terabytes isolados no processamento.

## O Problema
Tens dados imensos a entrar da arquitetura de base do teu Datalake. A requisição formal exige relatórios em tempo nos cubos de OLAP, resumos estatísticos intensivos baseados em durações e agregações ao invés de pings diários. Agarraste na framework, fizeste os grupos na memória primária, mandaste arrancar. Minutos depois recebes um som audível fatal de alarme ("Out of Memory") pois tentaste alocar conjuntos díspares na mesma box unitária de RAM. O teu dataset isolado excedeu os portões aceitáveis nos requisitos do hardware alocado.

## A Solução
Perante o excedente é preciso instanciar as funções da estratégia universal focada em fragmentar: o Distributed Aggregator (assente nos velhos ideais imortais do MapReduce). Configura-se de forma expedita nas engines e sub-sistemas de orquestrações de bases usando operações como agregados após a declaração funcional das lógicas de agrupamentos (ex. GROUP BY ... SUM ...). Por baixo do tapete, para se esquivar de processar fisicamente e explodir limites aloca partições pelas múltiplas redes das instâncias escravas subjacentes (clusters) forçando, a seu momento fatal, as custosas mas necessárias passagens cruzadas da reatribuição universal chamada `Shuffle`. Aqui se trocam entre a base totalidade apenas parcelas de subtotais e as somas resultam conjuntas de múltiplas partes combinadas das redes.

## Prós e Contras
- **Prós:** 
  - A forma essencial padronizada pelas plataformas que garante escalabilidade teórica sem horizontes de paragens desde que instâncias adjacentes em cloud colmatem.
  - Universalmente programável e abstratamente impercetível da API que simula um processo singular para o utilizador subjacente (código inalterável para GBs ou PBs).
- **Contras:** 
  - Pode tornar-se na infraestrutura e despesa "assassina principal do orçamento e de latência", induzindo fortes paragens ao forçar operações Shuffle via I/O da rede massivos.
  - Terrível suscetibilidade analítica associada a anomalias nefastas e desiquilíbrios acentuados ("Data Skews"); em domínios mal formados e assimétricos um nó escravo sofrerá atritos fatais (impondo técnicas defensivas laboriosas baseadas no 'Salting').