# Padrão: Gatilho Externo (External Trigger)

## 1. Resumo (O que é?)
O padrão de Gatilho Externo foca na arquitetura orientada a eventos para iniciar fluxos de dados de forma reativa. Ele determina que, em vez de o consumidor checar repetidamente se há novos dados (polling), o produtor envia ativamente uma notificação dizendo: "Ei, os dados chegaram, você já pode começar!".

## 2. O Problema
* Os seus dados (ex: liberações de novas funcionalidades ou referências de negócio) são produzidos com uma frequência irregular e imprevisível. Às vezes saem em 2 horas, às vezes levam dias.
* Seu sistema tenta descobrir dados novos rodando o mesmo script a cada hora ou através de *polling* constante. Isso gasta muitos recursos computacionais à toa quando nada mudou e cria sobrecarga de manutenção na orquestração.

## 3. A Solução
Ao adotar um Gatilho Externo, o modelo muda de *Pull* (puxar) para *Push* (empurrar). Envolve três etapas fundamentais: (1) O seu sistema se inscreve em um canal de notificações (um message bus, tópico ou lambda); (2) Quando os dados terminam na origem, a origem dispara o evento para esse canal; (3) Um serviço manipulador (handler) recebe essa mensagem no seu ecossistema e inicia programaticamente a execução da pipeline ou job de ingestão.

## 4. Consequências e Trade-offs
* **Vantagens:** Otimização máxima de recursos e custos, pois processamentos ocorrem estritamente quando há novidade. Arquitetura altamente responsiva.
* **Desvantagens/Atenção:** 
  * **Gestão de Erros Crítica:** A notificação passa a ser vital. Se a infraestrutura de mensageria perder a mensagem de disparo, seu pipeline pode nunca rodar para aquele dado. Recomenda-se tratar os erros rigidamente.
  * **Falta de Contexto (Pings Cegos):** Se o gatilho disser apenas "rode", sem detalhes (sem um payload como versão, ambiente, data de corte, ID do lote), o debug das execuções será incrivelmente difícil caso algo falhe. Sempre garanta que o evento possua os metadados necessários.

## 5. Exemplo de Aplicação Prática
O banco de dados transacional gera relatórios de fim de dia esporádicos (o fechamento pode atrasar ou adiantar). Assim que o dump está completo no bucket de armazenamento da nuvem (S3, por exemplo), o próprio bucket invoca automaticamente uma função serverless (AWS Lambda) que faz uma requisição HTTP REST para a API do seu Airflow ativando a rotina de limpeza.

## 6. Exemplo Simples de Código
```python
# Pseudocódigo de um handler Lambda engatilhando uma DAG no Airflow (via Push)
import requests

def lambda_handler(evento, contexto):
    arquivo_novo = evento['novo_arquivo_s3']
    
    resposta = requests.post(
        'http://airflow.empresa.com/api/v1/dags/meu-pipeline/dagRuns',
        json={'conf': {'arquivo_a_processar': arquivo_novo}},
        auth=('usuario', 'senha')
    )
    
    if resposta.status_code != 200:
        raise Exception("O disparo do pipeline falhou!")
```

## 7. Padrões Relacionados ou Nomes Similares
Padrão comum na filosofia *Event-Driven Architecture (EDA)* ou em abordagens modernas como *Data Contracts*. É uma alternativa direta (Push vs Pull) ao *Readiness Marker* e sua execução passiva através de Sensores.
