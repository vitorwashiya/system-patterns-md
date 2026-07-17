# Padrão: Sequenciador Isolado (Isolated Sequencer)

## 1. Resumo (O que é?)
O Sequenciador Isolado é um padrão focado na colaboração e orquestração entre fluxos de trabalho (pipelines) independentes que precisam colaborar para gerar um insight. Como as lógicas pertencem a domínios ou times diferentes, as pipelines não podem ser fundidas numa só, necessitando de uma forma estruturada para iniciar uma após a outra.

## 2. O Problema
* Seu time é dono do pipeline que extrai os dados, mas outro time cuida do pipeline que plota esses dados nos dashboards. 
* Vocês concordaram que não seria uma boa ideia colocar a montagem dos gráficos dentro do pipeline que processa dados.
* Como o time do painel saberá quando é a hora segura para o pipeline deles rodar?

## 3. A Solução
Determinar os limites de produção e consumo entre os pipelines fisicamente separados e estabelecer o acionamento entre eles. Existem duas abordagens principais: 
1. **Gatilho Baseado em Tarefa (Task-Based):** O pipeline produtor no final da sua rotina invoca diretamente (como um botão) a execução da pipeline consumidora.
2. **Gatilho Baseado em Dados (Data-Based):** O produtor simplesmente finaliza a sua gravação e deposita um arquivo sinalizador de "pronto" (*Readiness Marker*). O pipeline consumidor fica escutando a chegada deste arquivo e aciona a si mesmo, de modo fracamente acoplado.

## 4. Consequências e Trade-offs
* **Vantagens:** Mantém a autonomia de equipes, divisão de responsabilidades clara e evita pipelines gigantes inter-times.
* **Desvantagens/Atenção:** 
  * **Problemas de Desincronização de Agenda:** Se os pipelines compartilharem horários diferentes (produtor diário e consumidor por hora), lógicas extremamente complexas serão exigidas no acionador para impedir pulos de carga.
  * **Problemas de Comunicação:** É vital avisar os times dependentes sobre mudanças estruturais que possam corromper os gatilhos, pois como são isolados no código, os erros virão apenas em produção.

## 5. Exemplo de Aplicação Prática
O Time de Marketing possui um pipeline que prepara cupons. Para iniciar a distribuição, ele não roda sua lógica junto da criação. Assim que terminam o processo, o Airflow deles aciona um Sensor no servidor do Time do Aplicativo. O App percebe a finalização e liga sua própria pipeline enviando as mensagens push.

## 6. Exemplo Simples de Código
```python
# Abordagem acoplada baseada em tarefa em Apache Airflow
# A pipeline produtora usa um Operador externo para iniciar a consumidora
marcador_de_sucesso = ExternalTaskMarker(
    task_id='disparar_consumidores',
    external_dag_id='pipeline_geradora_de_agregados',
    external_task_id='sensor_gatilho_downstream'
)

# Acionador final
(carga_arquivos >> marcador_de_sucesso)
```

## 7. Padrões Relacionados ou Nomes Similares
Muitas das implementações confiam no uso de *Readiness Marker* (Marcador de Prontidão). Utiliza as abstrações de orquestração do *Local Sequencer*, mas focado no domínio organizacional (cross-pipeline).
