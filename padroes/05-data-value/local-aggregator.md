# Padrão: Agregador Local (Local Aggregator)

## 1. Resumo (O que é?)
O padrão de Agregador Local foi criado para executar aglomeração ou somatórios de dados localmente (dentro da mesma máquina de processamento), cortando drasticamente a dependência de transferências de dados pela rede (o temido *Shuffle*).

## 2. O Problema
* O Agregador Distribuído (*Distributed Aggregator*) gera muito tráfego de rede (Shuffle) em seu pipeline, tornando a agregação de dados ineficiente.
* No entanto, você sabe que seu armazenamento já gravou os dados "no lugar certo". Se todos os registros da "Chave A" já existem apenas no "Computador A", espalhar esses dados pela rede não tem justificativa lógica ou desempenho aceitável.

## 3. A Solução
Remover a troca pela rede. O padrão depende vitalmente que a estrutura do armazenamento de origem garanta que cada chave de grupo exista apenas em um espaço particionado fisicamente estático, de tal maneira que a máquina possa ler aquele espaço da partição e fazer a matemática inteira sozinha. Isso pode ser feito no nível do consumidor tirando proveito do particionamento ou em recursos nativos como *Bucketing* ou definindo chaves de distribuição como `DISTKEY` no AWS Redshift, de modo que dados similares vivam na mesma gaveta física de armazenamento de dados.

## 4. Consequências e Trade-offs
* **Vantagens:** Extrema velocidade de processamento com uso quase nulo de processamento em rede (Shuffle); elimina gargalos com "stragglers" (tarefas lentas isoladas afetando o grupo).
* **Desvantagens/Atenção:** 
  * **Problemas de Escalabilidade (Rebalanceamento Custoso):** Alterar o design base, como mudar de 10 partições nativas para 20, demanda a re-organização pesada e física de todo o armazenamento (Shuffle), pois você terá que "arrumar" os dados legados em novos buckets.
  * **Chaves Únicas Globalmente Exigidas:** Se diferentes consumidores quiserem agrupar pelos perfis de usuário, mas outros por produto, o *Local Aggregator* perderá eficácia para um deles, pois o sistema no armazenamento está agrupado para favorecer a chave escolhida na gravação.

## 5. Exemplo de Aplicação Prática
Em um modelo de streaming usando Kafka Streams, o tópico de compras foi previamente particionado na hora da escrita pelo ID do usuário. O aplicativo de fluxo agrupa os gastos lendo a partição diretamente através da memória compartilhada de forma contínua e imedia. Como os dados de um mesmo usuário sempre caem na mesma partição de leitura, a aplicação roda em ilhas independentes (isolated tasks) de altíssimo desempenho.

## 6. Exemplo Simples de Código
```sql
-- Em Bancos Colunares (ex: Redshift), declarar DISTKEY evita Shuffles nas agregações
CREATE TABLE visitas (
    visit_id INT,
    user_id INT,
    -- ...
) DISTSTYLE KEY DISTKEY(user_id); 
-- Consultas agrupadas por user_id se tornarão Locais
```

## 7. Padrões Relacionados ou Nomes Similares
Atua lado a lado com os padrões de organização de dados físicos como o *Bucket*. Oposto absoluto ao desempenho e uso de rede do *Distributed Aggregator*.
