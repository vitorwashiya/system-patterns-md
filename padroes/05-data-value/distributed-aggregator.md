# Padrão: Agregador Distribuído (Distributed Aggregator)

## 1. Resumo (O que é?)
O Agregador Distribuído tira proveito do processamento de frameworks massivamente paralelos para reunir itens isolados fisicamente sob lógicas similares (ex: agrupar vendas por país). É a base da geração de resumos em ambientes de "Big Data".

## 2. O Problema
* O seu conjunto de dados é tão grande que não cabe em uma única máquina. As linhas estão divididas em milhares de partições ao longo de uma arquitetura de Data Lake (ex: logs de visitas particionados por hora).
* O time de dados precisa gerar cubos analíticos ou visões de resumo (como contagens ou médias), cruzando dados que estão espalhados globalmente nesse repositório distribuído.

## 3. A Solução
Você utiliza um framework de processamento distribuído (como Spark ou um Data Warehouse como BigQuery). Sob o capô, esse padrão usa um comando de agrupamento (GROUP BY) seguido por uma função de redução (COUNT, AVG). Diferente de uma máquina única, os sistemas distribuídos realizam um embaralhamento (Shuffle) dos dados: as máquinas transferem as informações pela rede umas para as outras até que todos os dados referentes à mesma chave de agrupamento se encontrem fisicamente na mesma máquina (ou "nó"), permitindo que a agregação final aconteça. 

Quando possível, os nós primeiro executam computações locais (ex: contar quantas vendas existem naquele pedaço) antes de transmitirem a resposta para não sobrecarregar a rede.

## 4. Consequências e Trade-offs
* **Vantagens:** O único jeito eficiente de calcular estatísticas e métricas unificadas em bancos de dados ou Data Lakes imensos.
* **Desvantagens/Atenção:** 
  * **Tráfego de Rede (Shuffle):** O "Shuffle" é muitas vezes a operação mais lenta do cluster. Mover Giga/Terabytes de dados entre computadores leva tempo e custo de rede.
  * **Desequilíbrio de Dados (Data Skew):** Se uma chave de agregação (ex: o usuário "GamerFamoso") tiver milhões de registros a mais que um usuário comum, a máquina responsável por processar o "GamerFamoso" ficará travada (gargalo), enquanto o restante do cluster fica inativo esperando. Estratégias como o "Salting" (adicionar um sufixo aleatório) são usadas para mitigar isso.

## 5. Exemplo de Aplicação Prática
Para rodar a folha de pagamento de 1 milhão de funcionários globalmente, o Spark começa lendo os arquivos particionados geograficamente. As máquinas calculam o parcial nos continentes. Depois, realizam o "Shuffle", misturando os dados para calcular os agregados finais.

## 6. Exemplo Simples de Código
```python
# Spark DataFrame groupBy induz implicitamente um Distributed Aggregator
visitas_por_mes = (
    visitas_df
    .groupBy("mes_de_acesso")  # Provocará o Shuffle (rede)
    .count()                   # Aplica a Função de Agregação
)
```

## 7. Padrões Relacionados ou Nomes Similares
Implementa o clássico modelo *MapReduce* com as devidas otimizações modernas. Padrão contrastante ao *Local Aggregator* (quando a rede não precisa agir).
