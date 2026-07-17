# Padrão: Junção Alinhada (Aligned Fan-In)

## 1. Resumo (O que é?)
A Junção Alinhada (Aligned Fan-In) é um padrão que foca na fusão de diferentes ramos isolados de processamento em uma única tarefa convergente. Em um modelo alinhado, ele impõe uma restrição fundamental: *todas* as tarefas ou branches pai devem ser finalizados com absoluto sucesso antes do processamento comum continuar.

## 2. O Problema
* Seu pipeline consolida agregados diários a partir de registros separados por hora (24 arquivos diferentes de log).
* Não faz sentido varrer todas as lógicas complexas para os dados das 24 horas usando uma mesma máquina, pois o tempo se torna gargalo e a falha de 1 hora exigiria que você reprocessasse o dia todo (as 23 horas perfeitas seriam desperdiçadas). 

## 3. A Solução
Você usa recursos do orquestrador de dados (ou camada de processamento) para criar ramos de processamento paralelos, cada um tratando uma partição independente. Neste padrão de *Fan-In*, esses ramos fluem linearmente até convergirem para uma tarefa comum a todos no fim (ex: Agregação Final). Para ser "Alinhado", basta utilizar o mecanismo padrão do orquestrador, onde o passo que unifica só iniciará automaticamente se todas as execuções filhas antecedentes terminarem com sucesso (STATUS = SUCCESS).

## 4. Consequências e Trade-offs
* **Vantagens:** Otimiza brutalmente a performance (computação paralela). Beneficia lógicas de feedback: como pedaços pequenos são testados, erros ocorrem e notificam mais rápido do que um arquivo gigante processando tudo.
* **Desvantagens/Atenção:** 
  * **Pico de Infraestrutura:** Iniciar 24 jobs ao mesmo tempo requer capacidade de escalonamento elástico maciço na nuvem, ou forçará lentidão no agendamento.
  * **O Atraso pelo Mais Lento (Skew):** Como a junção alinhada trava até todo mundo terminar, se 23 tarefas rodarem em 1 minuto e a tarefa da 12ª hora demorar 3 horas por desvio de volume, todo o job aguardará as 3 horas para cruzar a barreira, engarrafando recursos.

## 5. Exemplo de Aplicação Prática
Para preparar o processamento salarial de empresas gigantes, o software gera tarefas paralelas rodando cálculo das filiais nos EUA, Brasil, Japão, etc. A tarefa que emite os comprovantes fica em *Fan-in* travado. Ela apenas rodará e emitirá o documento quando o cálculo de todas as filiais registrar status de completude; caso a do Brasil caia, nada é emitido.

## 6. Exemplo Simples de Código
```python
# O Apache Airflow adota Fan-In Alinhado por padrão
ramo_brasil = PythonOperator(task_id='carga_brasil')
ramo_eua = PythonOperator(task_id='carga_eua')
ramo_japao = PythonOperator(task_id='carga_japao')

agregador_final = PythonOperator(task_id='resultado_global')

# Três tarefas pai diferentes terminando em um único descendente (Fan-In)
[ramo_brasil, ramo_eua, ramo_japao] >> agregador_final
```

## 7. Padrões Relacionados ou Nomes Similares
Uma evolução direta para arquiteturas em lote massivamente divididas. Ele se contrapõe diretamente com a flexibilidade da restrição do *Unaligned Fan-In*.
