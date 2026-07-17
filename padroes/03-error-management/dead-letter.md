# Padrão: Fila de Mensagens Mortas (Dead-Letter)

## 1. Resumo (O que é?)
O padrão de Fila de Mensagens Mortas (Dead-Letter) consiste em capturar e separar registros que falharam durante o processamento em vez de interromper o fluxo principal. É uma válvula de escape para "mensagens venenosas" (poison pill messages) que permite que o sistema continue processando os dados válidos enquanto salva os problemáticos para análise futura.

## 2. O Problema
* Seu pipeline de streaming (ou batch) começa a falhar repetidamente porque a origem enviou registros malformados (dados inválidos ou corrompidos).
* O processamento segue a abordagem de falha rápida (fail-fast), derrubando a aplicação e exigindo que você reinicie tudo manualmente após corrigir ou pular o erro.

## 3. A Solução
Para não interromper o pipeline, você deve isolar os pontos onde o erro pode ocorrer (como lógicas de transformação customizadas) usando blocos `try-catch` ou funções seguras (que retornam `NULL` em caso de falha). Quando um registro inválido é detectado, ele é redirecionado para um armazenamento separado, conhecido como fila ou tabela de "Dead-Letter", que deve ser altamente disponível e fácil de monitorar (ex: buckets no S3, tópicos do Kafka). Opcionalmente, pode-se criar um pipeline secundário para reprocessar esses registros no futuro.

## 4. Consequências e Trade-offs
* **Vantagens:** O pipeline se torna resiliente a dados corrompidos; reduz o esforço de manutenção diária por falhas bobas.
* **Desvantagens/Atenção:** 
  * **Efeito Bola de Neve no Reprocessamento:** Se você optar por reprocessar esses dados no futuro (backfilling), pode acabar reenviando dados atrasados, obrigando todos os consumidores a jusante a reprocessarem seus dados também.
  * **Falsa Sensação de Sucesso:** Esconder os erros mantém o pipeline de pé, mas pode mascarar uma falha fatal. Você deve monitorar de perto a quantidade de erros gerados e criar alertas caso o volume de *dead-letters* ultrapasse um limite seguro.

## 5. Exemplo de Aplicação Prática
Um sistema recebe milhares de eventos de compras por segundo, mas de vez em quando o campo "moeda" vem com um valor desconhecido, quebrando a conversão matemática. Em vez de parar de processar as outras 99% compras válidas, o sistema anota as inválidas e as joga em um tópico do Kafka separado, onde a equipe analisa no dia seguinte para entender a falha na origem.

## 6. Exemplo Simples de Código
```python
# Lógica simples de Dead-Letter usando um try-catch
def processar_dados(registro):
    try:
        # Tenta processar o registro normalmente
        resultado = transformar(registro)
        salvar_no_destino_principal(resultado)
    except Exception as e:
        # Em caso de falha, anota o erro e envia para a Dead-Letter
        registro_com_erro = {"dado_original": registro, "erro": str(e)}
        salvar_na_dead_letter(registro_com_erro)
```

## 7. Padrões Relacionados ou Nomes Similares
Também conhecido como *Dead-Letter Queue (DLQ)*, ou *Side Outputs* em frameworks de streaming como Apache Flink.
