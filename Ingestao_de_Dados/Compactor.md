# Compactor

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Melhorar drasticamente a leitura e diminuir a latência do consumo ao transformar um vasto número de ficheiros muito pequenos de armazenamento em menos ficheiros de maiores dimensões, com as mesmas informações.

## O Problema
A pipeline de streaming está funcional, escrevendo registos na Silver layer quase a cada segundo. Alguns meses depois, o sistema torna-se problemático. Os trabalhos em batch a ler este repositório gastam **70% do tempo em operações morosas de listagem dos metadados dos milhares de micro-ficheiros** envolvidos e sofrem de problemas clássicos conhecidos por "small files problem", afetando desempenho e fatura em serviços na nuvem de tarifação por pedido e invocação (pay-as-you-go).

## A Solução
Para solucionar, usa-se uma tarefa programada extra, com o padrão Compactor, para compilar a multiplicidade de pequenos dados gerados a toda a hora em peças grandes e coesas de dados agrupados, reescrevendo semânticas para diminuir agressivamente o tamanho da lista global de ficheiros (overhead). Formatos baseados em tabela como Delta Lake efetuam isto usando `OPTIMIZE`, ao passo que outros frameworks combinam processos manuais de read-merge-write em tabelas particionadas.

## Prós e Contras
- **Prós:** 
  - Melhorias expressivas e inegáveis nos tempos de tempo-resposta dos sistemas de consumo.
  - Maior eficiência em queries de filtragem, e menos uso agressivo de memória na inicialização de Dataframes distribuídas.
- **Contras:** 
  - Compromisso de custo contra desempenho: a própria operação de compactação consome computação pesada que, se for muito regular, sairá cara.
  - O processo não se auto-limpa em alguns sistemas: pode obrigar a ações avulsas de "VACUUM" ou exclusão dos ficheiros lixo pequenos antigos.