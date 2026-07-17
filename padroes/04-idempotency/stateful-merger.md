# Padrão: Mesclador Com Estado (Stateful Merger)

## 1. Resumo (O que é?)
O Mesclador Com Estado (Stateful Merger) é uma variação avançada do padrão *Merger*. Ele armazena metadados extras (o "estado") que registram quais versões do conjunto de dados ou em quais períodos o merge aconteceu, viabilizando restaurações automáticas consistentes em cenários de reprocessamento (backfilling).

## 2. O Problema
* Ao usar o padrão *Merger* clássico para recalcular um dado processado dias atrás, a tabela mesclada principal fica corrompida.
* Ao re-injetar dados incrementais atrasados na linha do tempo, a ausência de controle do estado temporal causa inconsistência entre o que estava no momento do problema e o que foi inserido posteriormente pelas rodadas de merge subsequentes.

## 3. A Solução
Sempre que você realizar um *merge*, uma etapa posterior na orquestração anotará numa tabela secundária de controle (State Table) qual versão a tabela alvo assumiu naquele horário de execução (por exemplo, "execução 9h gerou a versão 5"). Caso a orquestração rode em modo "reprocessamento" para um momento do passado, uma tarefa inicial lerá a State Table, descobrirá a versão em que a tabela deveria estar na época, acionará o recurso de "Viagem no Tempo" (Time Travel) do banco de dados (revertendo a tabela para a versão passada) e só então aplicará o *merge* desejado.

## 4. Consequências e Trade-offs
* **Vantagens:** Oferece forte garantia de integridade e consistência durante o reprocessamento manual ou preenchimento histórico.
* **Desvantagens/Atenção:** 
  * **Armazenamentos Específicos:** Exige formatos de tabela modernos (Delta Lake, Iceberg) que deem suporte ao controle nativo e instantâneo de versões passadas (Time Travel e Restore). Do contrário, precisa simular recriando tabelas brutas enormes.
  * **Problemas com Vacuum (Limpeza):** Processos de limpeza (Vacuum) destroem fisicamente o histórico de versões. Se o histórico foi excluído por razões de economia, não será possível restaurar o estado nem rodar o backfilling para o período limpo.

## 5. Exemplo de Aplicação Prática
Para lidar com reclamações, o time de dados precisa refazer o cálculo de "saldos por loja" do feriado retrasado. No entanto, centenas de atualizações diárias de *merge* ocorreram desde então. Com o *Stateful Merger*, o job daquele dia do feriado roda no Airflow, automaticamente verifica o estado e diz: "Volte a tabela de saldos para a versão de segunda passada e só depois rode meu merge novamente, corrigindo meu dia".

## 6. Exemplo Simples de Código
```python
# Restaurando a versão através da API Python antes de rodar o merge
versao_do_passado = 4 # Valor consultado da Tabela de Estado

# A API do DeltaLake volta a tabela para a versão 4
DeltaTable.forName(spark, 'tabela_saldos').restoreToVersion(versao_do_passado)

# ... seguido de um processo clássico de MERGE do dado atualizado
```

## 7. Padrões Relacionados ou Nomes Similares
Uma progressão do padrão *Merger* projetada para sanar inconsistências. Fortemente ligado à arquitetura de *Time Travel* dos table formats.
