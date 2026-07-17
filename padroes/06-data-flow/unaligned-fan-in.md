# Padrão: Junção Desalinhada (Unaligned Fan-In)

## 1. Resumo (O que é?)
O padrão de Junção Desalinhada relaxa as restrições rígidas da arquitetura paralela, permitindo que a tarefa que consolida várias ramificações continue executando seu fluxo *mesmo se algumas* ou até mesmo *todas* as tarefas ramificadas anteriores tenham falhado.

## 2. O Problema
* Seu processamento consolida de hora em hora visitas para parceiros analíticos rodando de forma paralela via *Aligned Fan-In*.
* Entretanto, se a carga de 1 hora falhar pontualmente por falha no serviço remoto, toda a tabela do dia para de funcionar, prejudicando seus usuários analistas de negócio. 
* Em reunião com os stakeholders, a equipe concordou que é melhor entregar relatórios parciais (ex: resultados contendo apenas 23h processadas e ignorando o buraco falho de 1h) do que derrubar e bloquear tudo.

## 3. A Solução
Altere a infraestrutura do orquestrador relaxando as dependências. Nos orquestradores (como o Airflow), isso envolve ajustar o controle das regras de disparo (trigger_rules) da tarefa de consolidação final para algo flexível, como executar "após todos os pais apenas concluírem, seja qual for a conclusão" (ALL_DONE). Se quiser tratar erros em lote, orquestradores como Azure Data Factory ou AWS Step Functions utilizam blocos *Catch* que seguem em frente, anotando sucessos e falhas em uma lista de validação.

## 4. Consequências e Trade-offs
* **Vantagens:** Mantém a alta usabilidade do sistema para relatórios gerenciais focados em resiliência e não em bloqueios de falha restritiva.
* **Desvantagens/Atenção:** 
  * **Dados Parciais e Interpretação Errada:** Fornecer dados incompletos pode fazer com que a equipe de marketing pense que "As vendas caíram!" quando na verdade os dados de 3 filiais não subiram. É crítico gravar uma flag ou marcador visual (ex: `is_approximate = true`) na tabela final avisando que o dado do relatório é parcial.
  * **Legibilidade de Fluxo (Visualização):** Ao criar regras como "Rode o fluxo de Erro se o pai X quebrar, e o de Sucesso se der certo", os gráficos da orquestração começam a virar diagramas muito sujos, parecendo o padrão *Exclusive Choice*.

## 5. Exemplo de Aplicação Prática
Para rodar a compilação do "Resumo de Notícias do dia", o aplicativo dispara raspagem de dados em 10 jornais simultaneamente. Se o provedor de notícias 7 sair do ar com um *Time-Out*, a tarefa consolidadora aceita esse erro do jornal 7, avança na pipeline com os textos de 9 jornais e manda um e-mail de "Resumo" mesmo com parte em falta, deixando os leitores sempre com notícias na hora marcada.

## 6. Exemplo Simples de Código
```python
# Junção Desalinhada em Apache Airflow
from airflow.utils.trigger_rule import TriggerRule

consolidacao_parcial = PostgresOperator(
    task_id="atualiza_dashboard_geral",
    # Em vez do padrão ALL_SUCCESS, usamos ALL_DONE
    # Então, basta as filiais tentarem rodar para que a consolidação prossiga
    trigger_rule=TriggerRule.ALL_DONE
)

[filial_1, filial_2, filial_3] >> consolidacao_parcial
```

## 7. Padrões Relacionados ou Nomes Similares
Uma mutação que relaxa os termos do *Aligned Fan-In*. Muito relacionado à emissão de *Tolerâncias a Falhas* dos detectores de dados de *Error Management*.
