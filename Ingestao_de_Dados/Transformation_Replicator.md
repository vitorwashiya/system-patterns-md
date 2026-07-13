# Transformation Replicator

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Este padrão é a extensão do "Passthrough Replicator", cujo objetivo é replicar de forma semelhante os dados de produção para outros ambientes, mas garantindo ao mesmo tempo que são limpos de dados não conformes, confidenciais e protegidos por políticas.

## O Problema
Necessitas de testar uma nova versão da tua pipeline de processamento em ambiente de staging usando dados com formato real (para evitar surpresas no dia de release). Pela natureza volátil, ferramentas que simulam dados falham frequentemente. Seria bom copiar de produção, mas **o conjunto de dados inclui informações PII** que não têm autorização legal ou técnica para fluir para fora do ambiente restrito de produção.

## A Solução
Adiciona-se uma camada fina de transformação entre o momento de leitura do sistema de origem produtivo e o momento de escrita no armazenamento de staging. Isto pode assumir a forma de uma função customizada do Apache Spark, Flink ou uma query SQL explícita. A transformação exclui os campos sensíveis, renomeando ou utilizando técnicas de ocultamento PII, garantindo proteção de segurança e validade de formatos.

## Prós e Contras
- **Prós:** 
  - Garante o cumprimento de normativos legais (GDPR, CCPA), retendo a fidelidade operacional exigida ao teste de qualidade no staging.
  - Oferece um bom balanceamento de fidelidade versus privacidade de dados.
- **Contras:** 
  - Aumenta ativamente o risco associado à modificação de esquemas nos formatos de texto brutas, pois a conversão silenciosa imposta pode apagar acidentalmente formatações de timestamp originais.
  - Alto risco de dessincronização caso os campos que são classificados como privados venham a evoluir, podendo escapar controlos esquecidos pela equipa.