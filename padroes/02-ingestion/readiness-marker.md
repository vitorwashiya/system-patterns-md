# Padrão: Marcador de Prontidão (Readiness Marker)

## 1. Resumo (O que é?)
O padrão Marcador de Prontidão assegura que um processo de ingestão ou processamento só comece quando o conjunto de dados de origem estiver totalmente completo e disponível. O objetivo principal é impedir a leitura prematura de dados parcialmente processados.

## 2. O Problema
* Diversos pipelines consomem uma base de dados compartilhada, mas frequentemente eles tentam ler arquivos que ainda estão sendo gravados.
* Relatórios de Business Intelligence e modelos de Machine Learning (ML) ficam intermitentes e incorretos porque pegam o conjunto de dados "pela metade". Como não há conexão direta entre as tarefas de origens distintas, eles não sabem o momento certo de iniciar.

## 3. A Solução
Você pode indicar que o dado está pronto gravando um arquivo sinalizador de "sucesso" (uma *flag file*, como `_SUCCESS` ou `COMPLETED`) no local do armazenamento apenas após gravar todos os dados da carga daquele momento de forma bem-sucedida. O consumidor, então, monitora periodicamente o aparecimento desse arquivo marcador. Somente quando o arquivo é detectado, ele prossegue com a leitura dos dados reais.

## 4. Consequências e Trade-offs
* **Vantagens:** Desacopla dependências entre times (arquitetura Pull); evita processamentos que corrompem visualizações com dados incompletos.
* **Desvantagens/Atenção:** 
  * **Falta de Imposição Automática:** É uma convenção implícita. Um desenvolvedor desavisado pode configurar o consumidor para ignorar o marcador e ler o diretório direto, quebrando a regra. Exige forte comunicação entre os times.
  * **Dados Atrasados (Late Data):** Se um marcador for baseado em particionamento por tempo e for gerado (indicando fechamento da janela), e depois disso chegar um dado atrasado para aquela partição, os consumidores que já reagiram ao marcador podem nunca mais ler esse novo dado.

## 5. Exemplo de Aplicação Prática
Um pipeline noturno converte logs do formato JSON para Parquet. Este job leva 3 horas, atualizando diversas pastas de data. Após completar todas as pastas daquele dia, ele cria um arquivo vazio chamado `_SUCCESS` na raiz do dia correspondente. O orquestrador de Machine Learning do outro time tem um "sensor" rodando a cada 10 minutos procurando pela flag `_SUCCESS` de hoje para, então, iniciar o treinamento do modelo.

## 6. Exemplo Simples de Código
```python
# Exemplo de criação de uma task do Airflow para checar a existência do marcador
from airflow.sensors.filesystem import FileSensor

sensor_de_prontidao = FileSensor(
    task_id='esperar_pelo_success',
    filepath='/caminho/dos/dados/2024-01-01/_SUCCESS',
    mode='reschedule' # Para não travar recursos enquando espera
)
```

## 7. Padrões Relacionados ou Nomes Similares
Também conhecido na indústria como *Flag File*, *Success File* ou uso de *Sensores*. Tem relação conceitual e é o oposto do *External Trigger* (Gatilho Externo).
