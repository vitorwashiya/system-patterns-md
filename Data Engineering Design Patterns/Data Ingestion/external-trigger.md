
# External Trigger

## Classificação
- **Tipo:** Comportamental
- **Escopo:** Objeto

## Intenção
Substituir varreduras fúteis temporizadas agendando processamentos via estímulos remotos emitidos ativamente por entidades parceiras engatilhando reativamente execuções no momento de concepção precisa do dado.

## Também Conhecido Como
Event-Driven Data Ingestion, Push-based Trigger.

## Motivação (Contexto)
Sistemas base disparam emissões em regimes absurdamente esporádicos, impossíveis de serem adequadamente temporizados sob agendamentos clássicos. Rodar cronjobs arbitrários verificando periodicamente "novos dados" onera ociosamente orçamentos. Despacha-se ao emissor principal a governança restrita e dever afirmativo de notificar via rede à sua vontade originando orquestrações reativas de disparo de lote.

## Aplicabilidade
- Use o External Trigger quando:
  - Sistemas fornecedores geram e concretizam entregas de maneira não programada e intermitente.
  - Recursos na malha do orquestrador em cloud analítico não devem ser mantidos congelados permanentemente atuando poller ping ineficientes.
  - Semânticas puramente event-driven modernas ditam as regras arquitetônicas limitadas do data ingestion.

## Estrutura e Participantes
- **Event Publisher:** Emissor primário legatário de infraestrutura transacional originador do lote esporádico operando ativamente notificação externa de canal.
- **Notification Handler:** Interceptor transiente da infra operando de ponte leve, subscrito passivamente à escuta (Webhooks/Functions Serverless), formatando a ponte com os Orquestradores REST.
- **Orchestration Layer:** Receptor subjacente mantido em repouso passivo orquestrando execuções DAG pontuais unicamente após estimulação de injeção direta imperativa.

## Colaborações
Encerrado o preenchimento irregular do escopo, o Event Publisher dispara avisos em tópico. O Notification Handler interage, consumindo isoladamente este sinal com latência sub-segundo envelopando a ativação API. Este encaminha requisição estrita REST despertando o pipeline na Orchestration Layer inerte disparando as execuções isoladas reativas.

## Consequências
- **Prós:**
  - Evaporação dramática inegável do dispêndio de recursos "zumbis" passivos mitigando o overhead oneroso do pull-based.
  - Abordagem primorosa compatibilizadora com ecossistemas orientados amplamente a eventos modernos minimizando gargalos temporais.
- **Contras:**
  - Invocações ping frequentemente nuas destituídas da imersão em contexto restritivo local dificulta monitoramento minucioso no troubleshooting pós-falha de incidentes complexos.
  - Falhas catastróficas desamparadas e silenciadas (Dropped Events) em quedas atípicas da API de orquestração se não estiverem seguras amarradas ao resgate nativo de filas tolerantes transacionais (Dead-Letter queues).

## Implementação (Exemplo de Código)
```python
# Handler Cloud nativo encapsulado traduzindo webhook assíncrono em Invocação de Orquestrador Central
import json, requests

def trigger_handler_lambda(event, ctx):
    file_uri_target = extract_object_key_s3(event)
    payload_envelopado = { 'conf': { 'target_uri_load': file_uri_target } }

    # API POST Endpoint explícito estimulador
    trigger_exec = requests.post(
        'http://airflow-core:8080/api/v1/dags/esporadic-loader/dagRuns',
        data=json.dumps(payload_envelopado),
        auth=('internal_srv_usr', 'secret_pwd')
    )
    
    if trigger_exec.status_code != 200:
        raise Exception(f"Failed external trigger dispatching: {trigger_exec.text}")
        
    return True
```

## Padrões Relacionados

* **Readiness Marker:** Emissor clássico basilar oposto operando iterativamente que serve como contraste didático imponente à otimização promovida em arquiteturas inconstantes de tempo.
* **Dead-Letter:** Amarra estrutural fortíssima e necessária garantindo retenção de chamadas rejeitadas pela indisponibilidade transitória do orquestrador analítico final.

---
