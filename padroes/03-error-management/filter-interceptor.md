# Padrão: Interceptador de Filtros (Filter Interceptor)

## 1. Resumo (O que é?)
O Interceptador de Filtros é uma técnica para medir exatamente por que seus dados estão sendo descartados durante o processamento. Ele "intercepta" as lógicas de filtragem (comandos que removem registros que não atendem a certas regras) e contabiliza as ocorrências de descarte para ajudar no diagnóstico e na observabilidade do pipeline.

## 2. O Problema
* Seu trabalho de transformação descarta dados inválidos silenciosamente (usando um `filter()` ou uma cláusula `WHERE`).
* Repentinamente, você nota que o volume de dados filtrados saltou de 15% para 90%. 
* Como as ferramentas otimizam e colapsam múltiplas condições de filtro em uma só execução, fica quase impossível descobrir se o problema foi uma mudança no comportamento do usuário ou uma regressão de software no pipeline, sem investigar o dado no escuro.

## 3. A Solução
Em vez de usar filtros opacos, você deve encapsular (ou mapear) cada condição de filtro com um contador (como Accumulators no Spark ou somatórios em SQL). O Interceptador de Filtros avalia as condições: se for verdadeiro (passa no filtro), o dado segue; se for falso (não passa), um contador específico daquela regra é incrementado, e só então o dado é descartado. 

No final da execução, seu pipeline imprime ou salva o placar exato de cada filtro, fornecendo visibilidade total de "quantos registros morreram em qual regra".

## 4. Consequências e Trade-offs
* **Vantagens:** Proporciona extrema clareza e insights profundos sobre a qualidade do dado entrante e o comportamento do próprio código.
* **Desvantagens/Atenção:** 
  * **Impacto em Tempo de Execução:** Encapsular lógicas otimizadas nativas em contadores customizados pode adicionar sobrecarga (overhead) computacional, exigindo movimentação de contadores pela rede ou criações de tabelas temporárias adicionais em SQL.
  * **Dificuldade em Linguagens Declarativas:** Escrever isso em SQL puro é muito trabalhoso. Costuma exigir subqueries grandes com lógicas complexas de `CASE WHEN` para não perder dados, inflando o tamanho das queries.

## 5. Exemplo de Aplicação Prática
Um pipeline consolida dados de visitantes, filtrando "visitas curtas", "visitas sem IP" e "visitas de bots". Usando o interceptador, o desenvolvedor nota no log do final do dia que o filtro "visitas sem IP" foi responsável por 90% das perdas daquele dia, indicando rapidamente que a API externa de geolocalização deve estar fora do ar.

## 6. Exemplo Simples de Código
```python
# Lógica pseudo-codificada com Accumulators do Spark
contador_sem_id = spark.sparkContext.accumulator(0)

def filtro_interceptado(registro):
    if registro['user_id'] is None:
        contador_sem_id.add(1) # Contabiliza a falha específica
        return False           # Filtra o dado fora
    return True                # Mantém o dado válido

dados_limpos = dados_brutos.filter(filtro_interceptado)
```

## 7. Padrões Relacionados ou Nomes Similares
Atua na fronteira entre Engenharia de Dados e *Observabilidade de Dados*, fornecendo informações vitais que podem ser consumidas pelo *Online Observer* ou *Offline Observer*. Pode complementar o padrão *Dead-Letter* separando motivos lógicos em vez de só erros sistêmicos.
