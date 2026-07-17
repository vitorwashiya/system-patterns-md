# Padrão: Integrador Estático de Dados Atrasados (Static Late Data Integrator)

## 1. Resumo (O que é?)
O Integrador Estático de Dados Atrasados é uma estratégia de arquitetura em lote (batch) para recuperar dados valiosos que chegaram tarde demais. Ele faz isso processando sistematicamente, a cada nova execução, um período fixo e constante do passado.

## 2. O Problema
* Seu pipeline ignorou ou salvou dados atrasados usando o *Detector de Dados Atrasados*. 
* Embora a pontualidade seja prioridade, os dados atrasados têm valor comercial, e você precisa reintegrá-los ao conjunto de dados principal (por exemplo, atualizar os relatórios do dia anterior com vendas que não tinham caído no sistema).
* Você quer uma solução simples e fácil de programar para absorver isso sem complexidade extrema.

## 3. A Solução
Você define uma janela retrospectiva estática (lookback window). Se a janela for de 3 dias, a cada dia que o seu pipeline principal rodar, você instruirá o orquestrador a não processar apenas o dia de hoje, mas processar novamente e atualizar forçosamente os últimos 3 dias. Mesmo que não existam dados atrasados novos nesses dias, eles serão recalculados para cobrir a possibilidade.

## 4. Consequências e Trade-offs
* **Vantagens:** Muito fácil de codificar; garante alta qualidade final e absorve atrasos toleráveis com baixo esforço arquitetural.
* **Desvantagens/Atenção:** 
  * **Desperdício de Recursos:** Como a janela é estática, você reprocessará as partições dos últimos X dias mesmo se não tiver chegado nem um byte de dado atrasado, gastando computação à toa.
  * **Efeito Bola de Neve em Backfilling:** Se você for rodar um backfilling manual dos últimos 30 dias, por causa desse padrão, o sistema irá recriar os dias adjacentes repetidas vezes para cada dia da sua lista, criando sobreposições caras de processos.

## 5. Exemplo de Aplicação Prática
O relatório de visitas diárias roda à meia-noite gerando o resultado de hoje. O time de negócios exige precisão e aceita que os números fiquem flutuando e sejam corrigidos por até 2 dias. Então, a cada meia-noite, a pipeline recalcula o resultado de hoje, de ontem e de anteontem de forma incondicional.

## 6. Exemplo Simples de Código
```python
# Lógica do orquestrador (ex: Airflow) acionando jobs fixos retrospectivos
dias_para_voltar = 2
data_atual = '2024-01-03'

# Roda o dia atual e os 2 anteriores de forma fixa (Static)
for i in range(dias_para_voltar + 1):
    data_alvo = calcular_data(data_atual, menos_dias=i)
    # Lança as tarefas para: 2024-01-03, 2024-01-02, 2024-01-01
    disparar_pipeline_de_atualizacao(data_alvo)
```

## 7. Padrões Relacionados ou Nomes Similares
Também referenciado como *Fixed Lookback Window* (Janela Retrospectiva Fixa). Relaciona-se com o *Dynamic Late Data Integrator*, que busca curar as ineficiências de recursos desta abordagem estática.
