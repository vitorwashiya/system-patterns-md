# Padrão: Expurgador de Metadados (Metadata Purger)

## 1. Resumo (O que é?)
O padrão de Expurgador de Metadados aborda os problemas catastróficos que surgem não dos dados reais que sua empresa salva, mas sim do entupimento massivo de registros de "logs", "catálogos" e "tabelas de histórico de commits" acumuladas ao longo dos anos nos catálogos de motores Big Data modernos. O foco é destruir metadados inúteis.

## 2. O Problema
* Sua equipe usa Apache Hive Metastore ou formatadores Delta Lake/Iceberg. 
* Tudo parece mil maravilhosamente performático. Até que um ano depois, um simples `SELECT count(*)` no Iceberg começa a demorar longos 5 minutos para responder numa tabela que deveria ser veloz.
* O motor está perdendo horas escaneando e digerindo gigabytes de árvores de versões, histórico de transações passadas e caminhos de arquivos mortos (Time-Travel) antes sequer de encostar no dado real. 

## 3. A Solução
Como as estruturas de Data LakeHouse nunca excluem o histórico dos logs a cada `UPDATE` de forma automática, cabe ao engenheiro de dados programar pipelines de limpeza periódicas e compulsórias que executam as ações de `VACUUM` (expurgo físico de arquivos mortos sem referência no catálogo novo) ou "Poda de Logs" (limpar todas as transações que têm mais de 30 dias na tabela transacional). Ele foca os comandos não nos dados úteis dos clientes, mas na inteligência estrutural do banco.

## 4. Consequências e Trade-offs
* **Vantagens:** Essencial para salvar a viabilidade e performance de formatos analíticos em produção (Iceberg/Hudi/Delta). Otimiza não apenas o tempo de *plan (plano de query)* das ferramentas analíticas, mas economiza absurdos em custo de nuvem reduzindo arquivos esquecidos de metadados.
* **Desvantagens/Atenção:** 
  * **Viagem no Tempo Destruída:** Ao expurgar os históricos, a funcionalidade superestimada de "Time Travel" (Ex: Consultar a base como ela estava no Natal de 2022) é aniquilada para sempre. 
  * **Impactos Massivos de Leitura/Escrita Simultânea:** Rodar Expurgadores profundos muitas vezes exige que o sistema aplique cadeados invisíveis ou utilize recursos da rede imensos nas gavetas de metadados durante a limpeza, estressando servidores.

## 5. Exemplo de Aplicação Prática
Em um fluxo PySpark gerenciando Apache Iceberg, os desenvolvedores sabem que um job diário minúsculo faz atualizações (UPSERTS) pesadas. Se deixassem rodar por 3 anos, o Iceberg geraria milhões de versões minúsculas. Eles programam um job semanal no fim de semana rodando o expurgo de `snapshots`, reduzindo os logs e mantendo apenas a visão da última semana ativa.

## 6. Exemplo Simples de Código
```sql
-- Expurgador de Metadados em Ação no ecossistema Delta Lake / Iceberg

-- 1. Remove Snapshots inúteis das ramificações históricas do banco de dados 
CALL catalog.system.expire_snapshots('tabela_eventos', TIMESTAMP '2023-12-01 00:00:00.000');

-- 2. Limpa e varre as pastas na S3 de Arquivos mortos órfãos que o passo 1 largou
CALL catalog.system.remove_orphan_files(table => 'tabela_eventos');
```

## 7. Padrões Relacionados ou Nomes Similares
Funciona lado-a-lado com as tarefas de compressão sistêmica (Z-Ordering) ou *Structural Compactor*. Tem as mesmas ressalvas críticas do padrão *In-Place Overwriter*.
