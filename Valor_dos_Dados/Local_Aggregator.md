# Local Aggregator

**Categoria:** Valor dos Dados

## 🎯 Objetivo
Evitar constrangimentos nas sobrecargas nas redes internas ao induzir as interações baseadas nas formulações métricas conjuntas na sua raiz puramente confinada ao repositório alocado.

## O Problema
As execuções no "Distributed Aggregator" resolvem, mas tens constrições nos SLAs e não podes desperdiçar de forma banal nem a lentidão do engarrafamento que decorre das esperas associadas à permuta na infraestrutura interna, não aceitando, além disso, problemas causais onde "um nó lento no shuffle mata o tempo do resto dos escravos rápidos". Exiges acelerar por via natural as formulações conjuntas no espaço restritivo na sua partição alocada minimizando passos cruzados.

## A Solução
Como as partições nos subconjuntos lógicos deterministas prévios estáticos garantem localizações precisas em repositórios controlados antecipadamente podes usar a variante otimizada "Local Aggregator". Assim as funções em engines com suportes explícitos aplicam somas confinadas, reduzindo computações por partições na partição in-loco estrita (`groupByKey()` via streams da rede nativa de Apache Kafka streams com repartições explícitas). Assenta essencialmente a performance por depender inteiramente da fiabilidade imutável de origem pré-reorganizada como nas definições de buckets estritos na Cloud e tabelas orientadas via DDL a garantirem co-localidade inerte.

## Prós e Contras
- **Prós:** 
  - Eficiências titânicas nos recursos internos dispensando pesadas ações bloqueantes do shuffle e mantendo isolamentos inquebráveis dos workers a processar a fundo assíncrona.
  - Permite garantir que as partes que interagem respondem o mais ágil possível às emissões reativas contínuas.
- **Contras:** 
  - Demasiado refém estrutural do conceito do esquema subjacente; modificações estritas de negócio nas permutas nas partições obrigam as arquiteturas a congelamentos ou morosos processos inalteráveis de redistribuição da base passada.
  - Impõe forte dor de cabeça se consumidores precisarem relatar análises perante eixos da entidade originais distintos das lógicas em que foram acopladas no bucket originário, perdendo logo todas as vias ou obrigando o modelo principal Distributed a arrancar de igual modo.