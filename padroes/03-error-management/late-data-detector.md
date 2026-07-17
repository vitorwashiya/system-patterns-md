# Padrão: Detector de Dados Atrasados (Late Data Detector)

## 1. Resumo (O que é?)
O Detector de Dados Atrasados atua como um supervisor do tempo em pipelines de streaming e lotes baseados em eventos. Ele rastreia o progresso do "tempo do evento" (a hora em que o dado foi gerado) contra o "tempo de processamento" para identificar quais registros chegaram tarde demais ao sistema.

## 2. O Problema
* Em sistemas distribuídos e na internet móvel, alguns usuários podem perder conexão e enviar dados horas depois.
* O seu pipeline precisa fechar um cálculo (como agregar acessos por hora), mas ficar esperando indefinidamente pelo dado que "pode chegar" impossibilitaria o trabalho.
* Você precisa de uma regra clara para decidir o que considerar no fluxo atual e o que descartar ou processar separadamente como "atrasado".

## 3. A Solução
A solução envolve a definição de um marco temporal, conhecido em ferramentas de streaming como *Watermark* (marca d'água). Você determina que o pipeline rastreará sempre a maior (MAX) data/hora encontrada nos registros até o momento e aceitará uma tolerância fixa (ex: "aceito dados de até 30 minutos atrás"). Qualquer registro com a data original de criação mais antiga que essa marca d'água computada será classificado e detectado como atrasado.

## 4. Consequências e Trade-offs
* **Vantagens:** Permite que sistemas de dados continuem avançando na linha do tempo, processando o fluxo sem paralisar ou estourar a memória.
* **Desvantagens/Atenção:** 
  * **Problemas de Desequilíbrio (Skew):** Se uma parte do sistema enviar dados novos super rápido e outra enviar dados antigos importantes, a marca d'água global será empurrada pelo dado rápido e descartará os dados da fonte mais lenta. 
  * **E a Captura?** Apenas detectar não significa guardar. Algumas ferramentas descartam o dado atrasado sumariamente. Para não perder informações vitais, você precisará capturá-las e tratá-las em armazenamentos paralelos.

## 5. Exemplo de Aplicação Prática
Um jogo mobile manda a pontuação da fase para os servidores assim que o jogador vence. Se o jogador passar a fase no metrô sem sinal, o celular aguarda e envia a pontuação 40 minutos depois. O pipeline, configurado para esperar apenas 10 minutos (watermark de 10 min), classifica a pontuação do jogador como "atrasada" e não a processa no ranking principal ao vivo.

## 6. Exemplo Simples de Código
```python
# Configurando o Detector de Dados Atrasados via Watermark no PySpark
acessos_filtrados = (
    fluxo_acessos
    # Define que dados com mais de 30 minutos de atraso
    # em relação ao dado mais atual serão ignorados/marcados
    .withWatermark('horario_do_evento', '30 minutes')
    .groupBy(window('horario_do_evento', '5 minutes'))
    .count()
)
```

## 7. Padrões Relacionados ou Nomes Similares
Conhecido amplamente como *Watermarking*. Os registros atrasados identificados por este padrão frequentemente alimentam implementações como o *Static Late Data Integrator* e *Dynamic Late Data Integrator*.
