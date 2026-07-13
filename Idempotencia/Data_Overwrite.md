# Data Overwrite

**Categoria:** Idempotência

## 🎯 Objetivo
Possibilitar a regeneração de operações de substituição plena sem recorrer a metadados, operando nativamente ao nível do repositório de suporte físico, reescrevendo estritamente os resultados das camadas de dados de base.

## O Problema
A tua arquitetura utiliza um armazenamento em objeto ou motor sem mecanismos maduros de controlo transacional ou comandos facilitadores baseados em metadados como `TRUNCATE` (ex: Object Stores clássicos com ficheiros JSON simples). A solução anterior "Fast Metadata Cleaner" torna-se totalmente inviável pois requeres que as inserções não criem registos redundantes ou se espalhem por novos blocos gerando poluição invisível.

## A Solução
Perante a inacessibilidade a uma solução lógica, confias nas operações do plano nativo submetendo as rotinas que injetam a data ao padrão Data Overwrite. Isto pode consubstanciar-se em recursos programáticos diretos integrados em frameworks distribuídas (ex: opções `.mode('overwrite')` nos Spark writers) ou instruções SQL compactas e cirúrgicas que suportem sobrescritas como `INSERT OVERWRITE` em substituição da conjugação pesada `DELETE + INSERT`.

## Prós e Contras
- **Prós:** 
  - Funcionalidade que, dadas as raízes puras em dados, está mais vastamente disponível e é universal do que a eliminação via tabelas de metadata.
  - Elimina a dor de cabeça arquitetónica ligada a falhas onde as partições já guardam ecos erróneos.
- **Contras:** 
  - Custo associado ao Overhead IO nas instâncias; uma vez que iteramos ficheiros e discos massivamente para o varrimento, e não os apontadores da tabela.
  - Necessidade mandatória eventual de desencadear processos em background subsequentes de "VACUUM" ou purga física.