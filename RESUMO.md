# 📘 Guia de Estudo: Padrões de Engenharia de Dados

Este documento é seu material de estudo completo. Cada padrão é explicado em 4 tópicos essenciais: o **Problema** que ele resolve, a **Solução** arquitetural, as **Consequências** (trade-offs) e um **Exemplo Prático** com código. Para detalhes aprofundados, consulte o arquivo `.md` individual de cada padrão na pasta `padroes/`.

---

## 1. Ingestão de Dados

*Como movemos dados das origens para o nosso ambiente analítico.*

---

### 1.1 Full Loader (Carga Completa)
📄 `padroes/02-ingestion/full-loader.md`

**🔴 Problema:** Precisamos copiar uma tabela de referência (ex: lista de países, categorias de produtos) que não possui colunas de controle como `data_modificacao` para rastrear o que mudou.

**🟢 Solução:** Copiar a tabela inteira a cada execução. O pipeline faz `DROP` da tabela antiga no destino e a recria com os dados frescos da origem. Simples e eficaz para tabelas pequenas.

**⚖️ Consequências:**
- ✅ Simplicidade absoluta; sem lógica de rastreamento.
- ❌ Inviável para tabelas grandes (copiar 500 GB a cada hora é caro e lento).
- ❌ Janela de indisponibilidade: entre o DROP e a recriação, consumidores verão a tabela vazia.

**💻 Exemplo:**
```python
# PySpark — Full Loader simples
df = spark.read.jdbc(url="jdbc:postgresql://origem:5432/db", table="categorias")
df.write.mode("overwrite").saveAsTable("destino.categorias")
```

---

### 1.2 Incremental Loader (Carga Incremental)
📄 `padroes/02-ingestion/incremental-loader.md`

**🔴 Problema:** A tabela de vendas na origem tem 500 milhões de linhas. Copiar tudo a cada hora é impossível, mas a tabela possui uma coluna `data_atualizacao`.

**🟢 Solução:** Guardar a marca d'água (*watermark*) da última execução (ex: `2024-01-15 08:00:00`). Na próxima execução, ler apenas as linhas com `data_atualizacao > '2024-01-15 08:00:00'` e anexá-las (*append*) ao destino.

**⚖️ Consequências:**
- ✅ Rápido e barato: processa apenas o delta novo.
- ❌ Não captura exclusões físicas (*hard deletes*): se a origem apagou uma linha, o incremental nunca saberá.
- ❌ Depende de uma coluna confiável de rastreamento existir na origem.

**💻 Exemplo:**
```python
# PySpark — Incremental Loader com marca d'água
ultima_marca = spark.sql("SELECT MAX(data_atualizacao) FROM destino.vendas").collect()[0][0]
novos = spark.read.jdbc(url=url_origem, table="vendas",
    predicates=[f"data_atualizacao > '{ultima_marca}'"])
novos.write.mode("append").saveAsTable("destino.vendas")
```

---

### 1.3 Change Data Capture — CDC
📄 `padroes/02-ingestion/change-data-capture.md`

**🔴 Problema:** O negócio exige latência de segundos e precisa capturar exclusões físicas (*hard deletes*) que o Incremental Loader não detecta.

**🟢 Solução:** Ferramentas como Debezium escutam o *commit log* (diário de transações) do banco de dados. Cada `INSERT`, `UPDATE` ou `DELETE` é capturado como um evento e publicado num tópico Kafka em tempo real, sem consultar a tabela.

**⚖️ Consequências:**
- ✅ Latência de milissegundos; captura deletes.
- ❌ Complexidade operacional alta: requer Kafka, Debezium, Schema Registry.
- ❌ Mudanças de schema na origem podem quebrar o fluxo CDC se não forem tratadas.

**💻 Exemplo:**
```json
// Configuração Debezium (JSON) para PostgreSQL
{
  "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
  "database.hostname": "meu-banco.aws.com",
  "database.dbname": "vendas_db",
  "table.include.list": "public.vendas",
  "topic.prefix": "cdc-vendas"
}
```

---

### 1.4 Passthrough Replicator (Replicador de Passagem)
📄 `padroes/02-ingestion/passthrough-replicator.md`

**🔴 Problema:** Precisamos de uma cópia fiel (1:1) dos dados de produção em outro ambiente para debugging ou auditoria, sem alterar nada.

**🟢 Solução:** Copiar os arquivos bit a bit, preservando formato, metadados e estrutura original. Nenhuma transformação é aplicada.

**⚖️ Consequências:**
- ✅ Perfeito para ambientes de homologação e reprodução de bugs.
- ❌ Dados sensíveis (PII) são copiados sem proteção — risco de conformidade.

**💻 Exemplo:**
```bash
# AWS CLI — Cópia bruta entre buckets S3
aws s3 sync s3://producao/vendas/ s3://homologacao/vendas/ --exact-timestamps
```

---

### 1.5 Transformation Replicator (Replicador com Transformação)
📄 `padroes/02-ingestion/transformation-replicator.md`

**🔴 Problema:** Precisamos copiar dados de produção para testes, mas eles contêm CPFs e e-mails reais que não podem vazar.

**🟢 Solução:** Aplica uma camada de transformação *durante* a cópia. Os dados sensíveis são mascarados (ex: CPF vira `***.***.***-XX`) ou removidos antes de chegarem ao destino.

**⚖️ Consequências:**
- ✅ Conformidade com LGPD/GDPR garantida no destino.
- ❌ Mais lento que o Passthrough; lógica de mascaramento precisa ser mantida.

**💻 Exemplo:**
```python
# PySpark — Replicação com mascaramento de PII
df = spark.read.parquet("s3://producao/clientes/")
df_seguro = df.withColumn("cpf", lit("***.***.***-**")).drop("email")
df_seguro.write.parquet("s3://sandbox/clientes/")
```

---

### 1.6 Compactor (Compactador)
📄 `padroes/02-ingestion/compactor.md`

**🔴 Problema:** Streaming contínuo (Kafka → S3) gerou milhões de arquivos de 2 KB cada. Listar e abrir milhões de arquivos minúsculos é mais lento do que ler os dados em si (*Small Files Problem*).

**🟢 Solução:** Um job periódico lê os micro-arquivos acumulados e os funde em poucos arquivos grandes (ex: 128–256 MB cada), otimizando o I/O de leitura.

**⚖️ Consequências:**
- ✅ Leitura até 100x mais rápida após compactação.
- ❌ Consome recursos (CPU/memória) para ler e regravar os dados.
- ❌ Janela de risco: durante a compactação, leitores podem ver dados inconsistentes se não houver controle transacional.

**💻 Exemplo:**
```python
# PySpark — Compactação de micro-arquivos
df = spark.read.parquet("s3://lago/eventos/hora=14/")
df.coalesce(4).write.mode("overwrite").parquet("s3://lago/eventos/hora=14/")
```

---

### 1.7 Readiness Marker (Marcador de Prontidão)
📄 `padroes/02-ingestion/readiness-marker.md`

**🔴 Problema:** Um consumidor lê uma pasta S3 enquanto o produtor ainda está no meio da gravação, resultando em dados incompletos.

**🟢 Solução:** Ao final da gravação bem-sucedida, o produtor cria um arquivo-flag vazio (ex: `_SUCCESS`) na pasta. Consumidores só iniciam a leitura quando detectam esse arquivo.

**⚖️ Consequências:**
- ✅ Simples e universal; funciona com qualquer sistema de arquivos.
- ❌ Polling: consumidores precisam verificar repetidamente se o arquivo existe.

**💻 Exemplo:**
```python
# Airflow — FileSensor esperando o marcador
from airflow.sensors.filesystem import FileSensor
sensor = FileSensor(task_id="esperar_dados", filepath="s3://lago/vendas/_SUCCESS", poke_interval=60)
```

---

### 1.8 External Trigger (Gatilho Externo)
📄 `padroes/02-ingestion/external-trigger.md`

**🔴 Problema:** O pipeline acorda de hora em hora para checar se há arquivo novo no S3. Na maioria das vezes, não há nada — desperdiça recursos.

**🟢 Solução:** Inverte a lógica: o pipeline fica dormindo. Quando o arquivo chega no S3, o próprio S3 dispara uma notificação (webhook/evento) que acorda o pipeline sob demanda.

**⚖️ Consequências:**
- ✅ Zero desperdício de recursos; reação instantânea.
- ❌ Dependência de infraestrutura de eventos (SNS, EventBridge, Cloud Functions).
- ❌ Eventos podem se perder se o sistema receptor estiver fora do ar.

**💻 Exemplo:**
```python
# AWS Lambda acionada por evento S3
def handler(event, context):
    arquivo = event['Records'][0]['s3']['object']['key']
    # Dispara a DAG do Airflow via API
    requests.post("http://airflow/api/v1/dags/ingestao/dagRuns", json={"conf": {"arquivo": arquivo}})
```

---

## 2. Gerenciamento de Erros

*O que fazer quando dados ruins ou falhas acontecem no fluxo.*

---

### 2.1 Dead-Letter (Fila de Mensagens Mortas)
📄 `padroes/03-error-management/dead-letter.md`

**🔴 Problema:** Um registro corrompido (ex: JSON inválido) no meio de milhões faz o processamento inteiro falhar.

**🟢 Solução:** Envolve o processamento em `try/except`. Registros que causam erro são desviados para uma "fila morta" (tabela ou tópico separado). O resto continua normalmente.

**⚖️ Consequências:**
- ✅ Pipeline resiliente; nunca para por um registro ruim.
- ❌ Se a fila morta não for monitorada, erros sistêmicos passam despercebidos.
- ❌ Dados na fila morta exigem triagem e reprocessamento manual.

**💻 Exemplo:**
```python
# PySpark — Dead-Letter com try/except
for linha in lote:
    try:
        processar(linha)
    except Exception as e:
        salvar_dead_letter(linha, str(e))  # Desvia para tabela de erros
```

---

### 2.2 Windowed Deduplicator (Desduplicador em Janela)
📄 `padroes/03-error-management/windowed-deduplicator.md`

**🔴 Problema:** Falhas de rede fazem o app do cliente reenviar o mesmo evento de clique 3 vezes, inflando métricas.

**🟢 Solução:** Manter um cache de IDs processados dentro de uma janela de tempo (ex: últimos 10 minutos). Se um ID repetido chegar, é descartado silenciosamente.

**⚖️ Consequências:**
- ✅ Garante *exactly-once* na prática dentro da janela.
- ❌ Duplicatas que chegam *fora* da janela não serão detectadas.
- ❌ Memória necessária cresce com o volume de IDs únicos na janela.

**💻 Exemplo:**
```python
# Spark Structured Streaming — Deduplicação por janela
df_dedup = df.withWatermark("event_time", "10 minutes") \
    .dropDuplicates(["event_id", "event_time"])
```

---

### 2.3 Late Data Detector (Detector de Dados Atrasados)
📄 `padroes/03-error-management/late-data-detector.md`

**🔴 Problema:** Um sensor IoT ficou offline por 2 dias. Quando volta, envia eventos com timestamps do passado que bagunçam janelas de cálculo atuais.

**🟢 Solução:** Configurar *watermarks* — marcadores de tolerância temporal. Eventos com timestamp mais antigo que o watermark são classificados como "atrasados" e podem ser descartados ou tratados separadamente.

**⚖️ Consequências:**
- ✅ Protege a integridade das janelas analíticas atuais.
- ❌ Dados valiosos podem ser perdidos se a tolerância for muito rígida.

**💻 Exemplo:**
```python
# Spark Structured Streaming — Watermark de 1 hora
df.withWatermark("event_time", "1 hour") \
    .groupBy(window("event_time", "5 minutes")).count()
```

---

### 2.4 Static Late Data Integrator (Integrador Estático de Dados Atrasados)
📄 `padroes/03-error-management/static-late-data-integrator.md`

**🔴 Problema:** Dados importantes chegam com dias de atraso (ex: dados de parceiros). A tabela analítica do mês passado está incompleta.

**🟢 Solução:** O pipeline reprocessa automaticamente os últimos N dias a cada execução. Se o batch diário roda hoje, ele também reprocessa D-1, D-2 e D-3, incorporando qualquer dado atrasado que tenha aparecido.

**⚖️ Consequências:**
- ✅ Simples de implementar; garante que atrasos dentro da janela sejam absorvidos.
- ❌ Reprocessa partições desnecessariamente (mesmo que nada atrasado tenha chegado).
- ❌ Custo computacional multiplicado: 3 dias de lookback = 3x o custo diário.

**💻 Exemplo:**
```python
# Airflow — Reprocessamento dos últimos 3 dias
from datetime import timedelta
for i in range(3):
    data = execution_date - timedelta(days=i)
    reprocessar_particao(data)
```

---

### 2.5 Dynamic Late Data Integrator (Integrador Dinâmico de Dados Atrasados)
📄 `padroes/03-error-management/dynamic-late-data-integrator.md`

**🔴 Problema:** O integrador estático reprocessa 3 dias inteiros todos os dias. 99% das vezes, nenhum dado atrasado chegou — desperdício puro.

**🟢 Solução:** Antes de reprocessar, o pipeline consulta uma tabela de auditoria que registra *quais partições* receberam novos dados. Só reprocessa as partições efetivamente impactadas.

**⚖️ Consequências:**
- ✅ Economia brutal de recursos: processa só onde é necessário.
- ❌ Complexidade maior: exige uma tabela de auditoria e lógica condicional.

**💻 Exemplo:**
```sql
-- Descobrir quais partições receberam dados atrasados
SELECT DISTINCT particao_data FROM auditoria_chegada
WHERE data_registro > CURRENT_DATE - INTERVAL '1 day'
  AND particao_data < CURRENT_DATE - INTERVAL '1 day';
```

---

### 2.6 Filter Interceptor (Interceptador de Filtros)
📄 `padroes/03-error-management/filter-interceptor.md`

**🔴 Problema:** O pipeline filtra linhas com `idade < 0`, mas ninguém sabe que 30% dos registros estão sendo descartados silenciosamente.

**🟢 Solução:** Acoplar contadores a cada regra de filtro. A cada execução, registrar métricas: "Regra X rejeitou 5.000 linhas hoje" e publicar num dashboard.

**⚖️ Consequências:**
- ✅ Visibilidade total sobre a "sujeira" na origem.
- ❌ Overhead de logging e armazenamento dos relatórios de descarte.

**💻 Exemplo:**
```python
# PySpark — Contagem de rejeições por regra
total = df.count()
rejeitados = df.filter("idade < 0").count()
print(f"Regra 'idade_negativa': {rejeitados}/{total} rejeitados ({rejeitados/total*100:.1f}%)")
df_limpo = df.filter("idade >= 0")
```

---

### 2.7 Checkpointer (Ponto de Controle)
📄 `padroes/03-error-management/checkpointer.md`

**🔴 Problema:** O pipeline de streaming cai após processar 40 milhões de mensagens Kafka. Sem saber onde parou, ele reinicia do zero e duplica tudo.

**🟢 Solução:** Periodicamente salvar o *offset* (posição no tópico Kafka) numa base externa. Ao reiniciar, o pipeline consulta o último offset salvo e retoma exatamente de onde parou.

**⚖️ Consequências:**
- ✅ Recuperação rápida de falhas sem duplicação.
- ❌ Checkpoints muito frequentes geram I/O excessivo; muito espaçados geram reprocessamento.

**💻 Exemplo:**
```python
# Spark Structured Streaming — Checkpoint automático
df.writeStream \
    .option("checkpointLocation", "s3://checkpoints/streaming-vendas/") \
    .format("parquet").start("s3://lago/vendas/")
```

---

## 3. Idempotência

*Garantir que rodar o pipeline 1 vez ou 10 vezes produza o mesmo resultado correto, sem duplicar dados.*

---

### 3.1 Fast Metadata Cleaner (Limpador Rápido de Metadados)
📄 `padroes/04-idempotency/fast-metadata-cleaner.md`

**🔴 Problema:** Antes de regravar os dados de uma partição, precisamos apagar a partição antiga. Deletar linha por linha é absurdamente lento.

**🟢 Solução:** Usar comandos de DDL nativos do banco (`ALTER TABLE DROP PARTITION`, `TRUNCATE`) que removem partições inteiras em milissegundos atuando apenas nos metadados, sem varrer os dados.

**⚖️ Consequências:**
- ✅ Velocidade absurda: dropar uma partição de 1 TB leva milissegundos.
- ❌ Risco catastrófico: um erro no filtro da partição pode apagar dados errados irreversivelmente.

**💻 Exemplo:**
```sql
-- Hive/Spark SQL — Drop instantâneo de partição por metadados
ALTER TABLE vendas DROP IF EXISTS PARTITION (data='2024-01-15');
-- Agora reinsere os dados frescos nessa partição
INSERT INTO vendas PARTITION (data='2024-01-15') SELECT * FROM staging_vendas;
```

---

### 3.2 Data Overwrite (Sobrescrita de Dados)
📄 `padroes/04-idempotency/data-overwrite.md`

**🔴 Problema:** Relatórios diários gravados em arquivos numa pasta S3. Se o job rodar duas vezes, os arquivos duplicam.

**🟢 Solução:** Usar `mode("overwrite")` na escrita. Apaga os arquivos antigos da pasta e grava os novos no lugar. Se rodar 10 vezes, o resultado é idêntico.

**⚖️ Consequências:**
- ✅ Idempotência por destruição: simples e infalível.
- ❌ Janela de indisponibilidade entre o delete e a regravação.
- ❌ Não é adequado quando o destino contém dados de múltiplas origens na mesma pasta.

**💻 Exemplo:**
```python
# PySpark — Sobrescrita direta
df_novo.write.mode("overwrite").parquet("s3://lago/relatorios/dia=2024-01-15/")
```

---

### 3.3 Merger (Mesclador)
📄 `padroes/04-idempotency/merger.md`

**🔴 Problema:** A origem envia novos clientes *e* atualizações de clientes existentes. Um simples `append` duplicaria os existentes; um `overwrite` perderia o histórico.

**🟢 Solução:** UPSERT (MERGE): cruza o lote novo com a tabela existente pela chave primária. Linhas novas são inseridas; linhas existentes são atualizadas.

**⚖️ Consequências:**
- ✅ Mantém o histórico intacto e absorve atualizações.
- ❌ Custo alto: o MERGE precisa ler a tabela inteira para cruzar as chaves.
- ❌ Em tabelas muito grandes, o Shuffle do JOIN pode estourar memória.

**💻 Exemplo:**
```sql
-- Delta Lake / Spark SQL — MERGE (UPSERT)
MERGE INTO clientes AS alvo USING novos_clientes AS fonte
ON alvo.id = fonte.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

---

### 3.4 Stateful Merger (Mesclador com Estado)
📄 `padroes/04-idempotency/stateful-merger.md`

**🔴 Problema:** Em streaming, uma atualização atrasada chega *depois* de uma mais recente. O MERGE padrão sobrescreveria o dado novo com o antigo.

**🟢 Solução:** Manter um *state store* (memória de estado) que guarda o timestamp da última versão de cada chave. Só aceita a atualização se o timestamp do evento for mais recente que o estado atual.

**⚖️ Consequências:**
- ✅ Garante a ordem lógica dos eventos, mesmo fora de ordem.
- ❌ Consumo de memória cresce com o número de chaves ativas.
- ❌ Precisa de TTL para expirar chaves antigas e liberar memória.

**💻 Exemplo:**
```python
# Spark Structured Streaming — mapGroupsWithState
def atualizar_estado(chave, eventos, estado_anterior):
    for evento in sorted(eventos, key=lambda e: e.timestamp):
        if not estado_anterior.exists or evento.timestamp > estado_anterior.get().timestamp:
            estado_anterior.update(evento)
    return estado_anterior.get()
```

---

### 3.5 Keyed Idempotency (Idempotência por Chave)
📄 `padroes/04-idempotency/keyed-idempotency.md`

**🔴 Problema:** Falhas de rede fazem o app reenviar o mesmo pagamento. Sem controle, o cliente é cobrado duas vezes.

**🟢 Solução:** Gerar uma chave única determinística para cada evento (ex: `hash(usuario + produto + timestamp)`). O banco de destino usa essa chave como PRIMARY KEY — inserções duplicadas são rejeitadas automaticamente.

**⚖️ Consequências:**
- ✅ Idempotência delegada ao banco; zero lógica no código.
- ❌ Colisões de hash (raras) podem causar perda de dados legítimos.
- ❌ Custo de manutenção do índice de chaves em tabelas muito grandes.

**💻 Exemplo:**
```python
# Geração de chave determinística
import hashlib
chave = hashlib.md5(f"{usuario_id}_{produto_id}_{timestamp}".encode()).hexdigest()
# O banco rejeita silenciosamente se a chave já existir (INSERT IGNORE / ON CONFLICT DO NOTHING)
```

---

### 3.6 Transactional Writer (Escritor Transacional)
📄 `padroes/04-idempotency/transactional-writer.md`

**🔴 Problema:** O dashboard lê a tabela *durante* a escrita do pipeline. O resultado: gráficos mostram metade dos dados do dia.

**🟢 Solução:** Usar formatos transacionais (Delta Lake, Iceberg) que garantem ACID. A escrita só se torna visível para leitores após o commit final. Se o job falhar no meio, nada é publicado — atomicidade total.

**⚖️ Consequências:**
- ✅ Leitores nunca veem dados parciais ou inconsistentes.
- ❌ Overhead de metadados e logs transacionais.
- ❌ Requer formatos modernos (Delta/Iceberg) — não funciona com CSV/JSON simples.

**💻 Exemplo:**
```python
# Delta Lake — Escrita transacional (atômica)
df.write.format("delta").mode("overwrite") \
    .option("replaceWhere", "data = '2024-01-15'") \
    .save("s3://lago/vendas_delta/")
```

---

### 3.7 Proxy
📄 `padroes/04-idempotency/proxy.md`

**🔴 Problema:** Dados financeiros e legais nunca podem ser alterados (WORM — Write Once Read Many). Mas a diretoria precisa ver correções no painel.

**🟢 Solução:** O arquivo bruto histórico permanece intocável. Uma View (visão lógica) é colocada na frente como "vitrine". As correções são aplicadas apenas na camada da View, que une o histórico com uma tabela de correções na hora da leitura.

**⚖️ Consequências:**
- ✅ Histórico bruto 100% preservado para auditoria.
- ❌ Complexidade de manutenção: duas tabelas (bruta + correções) precisam ser gerenciadas.
- ❌ Leitura mais lenta por exigir JOIN na hora da consulta.

**💻 Exemplo:**
```sql
-- View Proxy que aplica correções sobre o histórico imutável
CREATE VIEW vendas_corrigidas AS
SELECT COALESCE(c.valor, h.valor) AS valor, h.produto, h.data
FROM historico_bruto h LEFT JOIN correcoes c ON h.id = c.id;
```

---

## 4. Valor dos Dados

*Transformar dados brutos em inteligência analítica real.*

---

### 4.1 Static Joiner (Junção Estática)
📄 `padroes/05-data-value/static-joiner.md`

**🔴 Problema:** A tabela de vendas tem `produto_id`, mas o painel precisa mostrar o `nome_produto`. Essa informação está numa tabela separada.

**🟢 Solução:** JOIN clássico entre a tabela de fatos (vendas) e a tabela de dimensão (produtos) pela chave comum `produto_id` durante o processamento batch.

**⚖️ Consequências:**
- ✅ Enriquecimento direto com dados de referência.
- ❌ Se a tabela de dimensão for enorme, o Shuffle no JOIN distribuído pode ser caro.

**💻 Exemplo:**
```python
# PySpark — JOIN estático entre fatos e dimensão
df_enriquecido = df_vendas.join(df_produtos, "produto_id", "left")
```

---

### 4.2 Dynamic Joiner (Junção Dinâmica)
📄 `padroes/05-data-value/dynamic-joiner.md`

**🔴 Problema:** Em streaming, o clique do usuário e o pagamento são eventos separados que chegam em momentos diferentes. Precisamos correlacionar os dois.

**🟢 Solução:** Reter ambos os streams em uma janela temporal (ex: 10 minutos). Se o clique e o pagamento do mesmo `session_id` chegarem dentro da janela, o JOIN acontece.

**⚖️ Consequências:**
- ✅ Correlação em tempo real entre fluxos independentes.
- ❌ Pares que não se encontrarem na janela são perdidos.
- ❌ Memória alta: ambos os lados do JOIN ficam retidos.

**💻 Exemplo:**
```python
# Spark Streaming — JOIN temporal entre dois streams
cliques.join(pagamentos,
    expr("cliques.session_id = pagamentos.session_id AND "
         "pagamentos.event_time BETWEEN cliques.event_time AND cliques.event_time + interval 10 minutes"))
```

---

### 4.3 Wrapper (Envoltório)
📄 `padroes/05-data-value/wrapper.md`

**🔴 Problema:** A API do parceiro retorna JSONs complexos e instáveis que mudam de formato sem aviso, quebrando o pipeline.

**🟢 Solução:** Guardar o JSON bruto inteiro numa coluna "envelope" (`raw_payload`) e extrair apenas campos estáveis (ID, timestamp) como colunas do cabeçalho. Se o formato interno mudar, o cabeçalho sobrevive.

**⚖️ Consequências:**
- ✅ Resiliência total contra mudanças no schema da origem.
- ❌ Consultas sobre o conteúdo do envelope exigem parsing em tempo de leitura.

**💻 Exemplo:**
```python
# PySpark — Wrapper guardando payload bruto
df = df.withColumn("raw_payload", to_json(struct("*"))) \
       .select("id", "timestamp", "raw_payload")
```

---

### 4.4 Metadata Decorator (Decorador de Metadados)
📄 `padroes/05-data-value/metadata-decorator.md`

**🔴 Problema:** Colunas técnicas de engenharia (`pipeline_id`, `data_ingestao`, `versao_schema`) poluem a visão dos analistas de negócios.

**🟢 Solução:** Injetar os metadados técnicos em colunas separadas prefixadas (ex: `_meta_pipeline_id`) ou aninhadas num struct `_metadata`, mantendo-as ocultas da visão padrão dos analistas.

**⚖️ Consequências:**
- ✅ Analistas veem tabelas limpas; engenheiros têm rastreabilidade completa.
- ❌ Overhead de armazenamento para as colunas extras.

**💻 Exemplo:**
```python
# PySpark — Decorando com metadados de engenharia
df = df.withColumn("_meta_pipeline_id", lit("vendas_v3")) \
       .withColumn("_meta_ingestao_ts", current_timestamp())
```

---

### 4.5 Distributed Aggregator (Agregador Distribuído)
📄 `padroes/05-data-value/distributed-aggregator.md`

**🔴 Problema:** Calcular a média de 300 bilhões de registros é impossível num único servidor.

**🟢 Solução:** Distribuir o cálculo em centenas de máquinas (Map): cada uma calcula a soma parcial dos seus registros locais. Um passo final (Reduce) consolida as somas parciais no resultado global.

**⚖️ Consequências:**
- ✅ Escala horizontalmente sem limite teórico.
- ❌ Shuffle na rede: mover dados entre nós é caro e pode ser o gargalo.

**💻 Exemplo:**
```python
# PySpark — Agregação distribuída (o Spark faz o Map-Reduce internamente)
df.groupBy("cidade").agg(avg("valor"), count("*"))
```

---

### 4.6 Local Aggregator (Agregador Local)
📄 `padroes/05-data-value/local-aggregator.md`

**🔴 Problema:** Enviar bilhões de linhas individuais pela rede para o nó central do agrupamento estrangula a banda.

**🟢 Solução:** Cada máquina agrega localmente antes de enviar. Se o nó tem 1 milhão de linhas de SP, ele envia apenas 1 linha: `{SP: soma=X, count=Y}`. O tráfego de rede cai drasticamente.

**⚖️ Consequências:**
- ✅ Reduz o Shuffle em ordens de magnitude.
- ❌ Só funciona para operações associativas e comutativas (soma, contagem, max/min). Não funciona para mediana.

**💻 Exemplo:**
```python
# PySpark — combineByKey faz a agregação local automática
rdd.combineByKey(createCombiner, mergeValue, mergeCombiners)
```

---

### 4.7 Incremental Sessionizer (Sessionizador Incremental)
📄 `padroes/05-data-value/incremental-sessionizer.md`

**🔴 Problema:** Entender quanto tempo cada usuário ficou ativo no site. Os logs estão fragmentados em partições diárias.

**🟢 Solução:** No final de cada dia, o batch agrupa os eventos por `user_id`, ordena por timestamp e calcula a duração da sessão. Sessões que cruzam a meia-noite são tratadas reconectando com a partição do dia anterior.

**⚖️ Consequências:**
- ✅ Funciona bem em batch com dados históricos.
- ❌ Sessões que cruzam partições exigem lógica extra de "costura".

**💻 Exemplo:**
```python
# PySpark — Sessionização diária por janela de inatividade de 30 min
from pyspark.sql.functions import lag, unix_timestamp
df = df.withColumn("gap", unix_timestamp("ts") - lag(unix_timestamp("ts")).over(user_window))
df = df.withColumn("nova_sessao", when(col("gap") > 1800, 1).otherwise(0))
```

---

### 4.8 Stateful Sessionizer (Sessionizador com Estado)
📄 `padroes/05-data-value/stateful-sessionizer.md`

**🔴 Problema:** Precisamos detectar o fim de uma sessão *em tempo real* para disparar ações imediatas (ex: enviar e-mail de abandono de carrinho).

**🟢 Solução:** O motor de streaming mantém em memória o estado de cada sessão ativa. Se nenhum evento do usuário chega em N minutos, um timer dispara e fecha a sessão, emitindo o resultado.

**⚖️ Consequências:**
- ✅ Reação em tempo real ao término da sessão.
- ❌ Consumo alto de memória para manter estado de milhões de sessões simultâneas.

**💻 Exemplo:**
```python
# Spark Streaming — Session Window nativa
df.groupBy(session_window("event_time", "30 minutes"), "user_id").count()
```

---

### 4.9 Bin Pack Orderer (Empacotador em Lotes)
📄 `padroes/05-data-value/bin-pack-orderer.md`

**🔴 Problema:** A API de destino aceita no máximo 50 MB por requisição. Enviar item por item é lento; enviar tudo de uma vez estoura o limite.

**🟢 Solução:** Agrupar os itens em pacotes que se aproximem do limite máximo (50 MB) sem ultrapassá-lo, otimizando o throughput.

**⚖️ Consequências:**
- ✅ Máxima eficiência de rede e throughput.
- ❌ Lógica de empacotamento adiciona complexidade ao código.

**💻 Exemplo:**
```python
# Python — Empacotamento respeitando limite de tamanho
pacote, tamanho = [], 0
for item in itens:
    if tamanho + len(item) > LIMITE_50MB:
        enviar_api(pacote); pacote, tamanho = [], 0
    pacote.append(item); tamanho += len(item)
```

---

### 4.10 FIFO Orderer (Ordenador FIFO)
📄 `padroes/05-data-value/fifo-orderer.md`

**🔴 Problema:** Em processamento distribuído, um `DELETE` pode ser executado *antes* do `INSERT` correspondente, corrompendo o estado.

**🟢 Solução:** Forçar processamento em fila única (First-In, First-Out). Quem chegou primeiro é processado primeiro. Usa partições Kafka com chave fixa por entidade para garantir ordem.

**⚖️ Consequências:**
- ✅ Garante ordem absoluta das operações.
- ❌ Elimina paralelismo: uma única fila se torna gargalo de throughput.

**💻 Exemplo:**
```python
# Kafka Producer — Garantir FIFO por chave de partição
producer.send("topico", key=b"usuario_123", value=evento)
# Todos os eventos do usuario_123 vão para a mesma partição → ordem garantida
```

---

## 5. Fluxo de Dados

*Como orquestrar e encadear tarefas de forma inteligente.*

---

### 5.1 Local Sequencer (Sequenciador Local)
📄 `padroes/06-data-flow/local-sequencer.md`

**🔴 Problema:** Um código monolítico de 2.000 linhas falha. A mensagem de erro não diz em qual etapa lógica o problema ocorreu.

**🟢 Solução:** Dividir o monólito em tarefas sequenciais nomeadas (Task 1 → Task 2 → Task 3) no orquestrador. Se a Task 2 falhar, o log aponta exatamente "Task 2: erro na linha X".

**⚖️ Consequências:**
- ✅ Debug cirúrgico; reexecução parcial (só da task que falhou).
- ❌ Overhead de orquestração para tarefas muito simples.

**💻 Exemplo:**
```python
# Airflow — DAG sequencial
with DAG("pipeline_vendas") as dag:
    extrair >> transformar >> validar >> carregar
```

---

### 5.2 Isolated Sequencer (Sequenciador Isolado)
📄 `padroes/06-data-flow/isolated-sequencer.md`

**🔴 Problema:** A equipe de Analytics quer rodar sua DAG só quando a DAG de ingestão do time de Data Engineering terminar com sucesso. Mas são repositórios e Airflows separados.

**🟢 Solução:** Usar *Sensors* (sensores) que ficam verificando o estado de uma task/DAG externa. Quando o sensor confirma "verde", a DAG local inicia.

**⚖️ Consequências:**
- ✅ Autonomia total entre equipes; desacoplamento de deploys.
- ❌ Polling do sensor consome slots do scheduler enquanto espera.

**💻 Exemplo:**
```python
# Airflow — ExternalTaskSensor aguardando outra DAG
sensor = ExternalTaskSensor(
    task_id="esperar_ingestao", external_dag_id="dag_ingestao",
    external_task_id="carregar", poke_interval=120)
```

---

### 5.3 Aligned Fan-In (Convergência Alinhada)
📄 `padroes/06-data-flow/aligned-fan-in.md`

**🔴 Problema:** O relatório consolidado precisa de dados de 3 fontes. Se uma falhar, o relatório sai incompleto e enganoso.

**🟢 Solução:** O orquestrador exige que TODAS as 3 tarefas predecessoras terminem com sucesso antes de liberar a task seguinte. Se qualquer uma falhar, tudo para.

**⚖️ Consequências:**
- ✅ Garante integridade total dos dados consolidados.
- ❌ A fonte mais lenta dita o ritmo de todo o pipeline (gargalo).

**💻 Exemplo:**
```python
# Airflow — Trigger Rule all_success (padrão)
[fonte_A, fonte_B, fonte_C] >> relatorio_consolidado
```

---

### 5.4 Unaligned Fan-In (Convergência Desalinhada)
📄 `padroes/06-data-flow/unaligned-fan-in.md`

**🔴 Problema:** O dashboard tolera dados parciais. Travar tudo porque uma fonte secundária falhou é pior do que exibir dados incompletos.

**🟢 Solução:** Usar `trigger_rule="none_failed"` ou `"all_done"`. A task seguinte roda mesmo que algumas predecessoras tenham sido puladas ou falhado.

**⚖️ Consequências:**
- ✅ Alta disponibilidade do pipeline; nunca para.
- ❌ Risco de publicar dados incompletos sem que o consumidor saiba.

**💻 Exemplo:**
```python
# Airflow — Convergência tolerante a falhas
relatorio = PythonOperator(task_id="relatorio", trigger_rule="none_failed")
[fonte_A, fonte_B, fonte_C] >> relatorio
```

---

### 5.5 Parallel Split (Divisão Paralela)
📄 `padroes/06-data-flow/parallel-split.md`

**🔴 Problema:** Três dashboards consultam o mesmo Data Warehouse pesado separadamente, triplicando o custo de varredura.

**🟢 Solução:** Carregar a fonte uma única vez para um cache temporário. As 3 tasks paralelas leem do cache em vez da fonte original.

**⚖️ Consequências:**
- ✅ Custo de leitura da fonte reduzido a 1x; paralelismo máximo.
- ❌ O cache precisa de espaço temporário e lógica de limpeza.

**💻 Exemplo:**
```python
# Airflow — Um produtor alimenta N consumidores paralelos
carregar_cache >> [dashboard_vendas, dashboard_marketing, dashboard_financeiro]
```

---

### 5.6 Exclusive Choice (Escolha Exclusiva)
📄 `padroes/06-data-flow/exclusive-choice.md`

**🔴 Problema:** O pipeline do Brasil não deve executar a lógica do Peru. Rodar ambas gasta CPU desnecessariamente.

**🟢 Solução:** Usar `BranchPythonOperator` no Airflow. Uma função avalia a condição (ex: `país == "BR"`) e retorna o `task_id` da ramificação correta. As outras são puladas (`SKIP`).

**⚖️ Consequências:**
- ✅ Zero desperdício: só executa o caminho relevante.
- ❌ Complexidade de manutenção cresce com o número de ramificações.

**💻 Exemplo:**
```python
# Airflow — BranchPythonOperator
def escolher_pais(**ctx):
    return "pipeline_br" if ctx["params"]["pais"] == "BR" else "pipeline_pe"
branch = BranchPythonOperator(task_id="escolha", python_callable=escolher_pais)
```

---

### 5.7 Single Runner (Execução Única)
📄 `padroes/06-data-flow/single-runner.md`

**🔴 Problema:** O pipeline das 8h atrasa e ainda está rodando quando a execução das 9h inicia. Duas instâncias paralelas tentam escrever na mesma tabela, causando corrupção.

**🟢 Solução:** Configurar `max_active_runs=1` no orquestrador. A segunda instância fica na fila esperando a primeira terminar.

**⚖️ Consequências:**
- ✅ Elimina conflitos de escrita simultânea.
- ❌ Se uma execução travar, todas as seguintes ficam enfileiradas indefinidamente.

**💻 Exemplo:**
```python
# Airflow — DAG com execução única
dag = DAG("pipeline_critico", max_active_runs=1, catchup=False)
```

---

### 5.8 Concurrent Runner (Execução Concorrente)
📄 `padroes/06-data-flow/concurrent-runner.md`

**🔴 Problema:** Precisamos recalcular (backfill) 365 dias de dados. Com `max_active_runs=1`, levaria 365 execuções sequenciais.

**🟢 Solução:** Aumentar `max_active_runs` para N (ex: 16). O orquestrador dispara 16 execuções simultâneas, utilizando toda a capacidade do cluster.

**⚖️ Consequências:**
- ✅ Backfill 16x mais rápido.
- ❌ Só funciona se as execuções forem genuinamente independentes (sem dependência temporal).
- ❌ Pode sobrecarregar o cluster e o banco de dados de destino.

**💻 Exemplo:**
```python
# Airflow — DAG com concorrência alta para backfill
dag = DAG("backfill_historico", max_active_runs=16, catchup=True)
```

---

## 6. Segurança de Dados

*Proteger informações sensíveis e obedecer às leis de privacidade.*

---

### 6.1 Vertical Partitioner — Segurança
📄 `padroes/07-data-security/vertical-partitioner-security.md`

**🔴 Problema:** A LGPD obriga a deletar todos os dados do "João". Mas o CPF e nome do João estão espalhados em bilhões de linhas de logs de telemetria.

**🟢 Solução:** Na ingestão, separar os dados em duas tabelas: **Identidade** (CPF, nome, e-mail — 1 linha por usuário) e **Eventos** (cliques, acessos — bilhões de linhas com apenas um `id_cego`). Para deletar o João, basta apagar 1 linha na tabela de Identidade; o `id_cego` nos eventos perde o vínculo humano.

**⚖️ Consequências:**
- ✅ Exclusão LGPD/GDPR em milissegundos em vez de horas.
- ❌ Consultas que precisam reunir identidade + eventos exigem JOIN pela rede.
- ❌ Complexidade de ingestão: dois destinos em vez de um.

**💻 Exemplo:**
```python
# Spark — Separação vertical na ingestão
eventos = dados.drop("nome", "cpf", "email")
eventos.write.save("s3://logs_brutos/")
identidade = dados.select("id_cliente", "nome", "cpf", "email").dropDuplicates()
identidade.write.save("s3://dados_pessoais/")
```

---

### 6.2 In-Place Overwriter (Sobrescritor No Local)
📄 `padroes/07-data-security/in-place-overwriter.md`

**🔴 Problema:** O Data Lake legado não foi projetado com separação vertical. Os dados pessoais estão misturados em todas as linhas de todas as tabelas.

**🟢 Solução:** Buscar e destruir: `DELETE FROM tabela WHERE usuario='João'`. Em Delta/Iceberg, o motor reescreve os arquivos afetados sem a linha do João. Seguido de `VACUUM` para remover os arquivos antigos fisicamente.

**⚖️ Consequências:**
- ✅ Funciona em qualquer arquitetura, mesmo legada.
- ❌ Custo brutal de I/O: lê e reescreve arquivos inteiros para deletar 1 linha.
- ❌ O `VACUUM` é obrigatório — sem ele, os arquivos velhos (com o dado do João) continuam no disco.

**💻 Exemplo:**
```sql
-- Delta Lake — Delete + Vacuum
DELETE FROM clientes_legado WHERE cliente_id = 'ABCD-1234';
VACUUM clientes_legado RETAIN 0 HOURS;
```

---

### 6.3 Fine-Grained Accessor for Tables
📄 `padroes/07-data-security/fine-grained-accessor-tables.md`

**🔴 Problema:** 30 vendedores acessam a mesma tabela de vendas. Cada um deveria ver apenas suas próprias vendas. Criar 30 tabelas separadas é inviável.

**🟢 Solução:** Ativar *Row Level Security* (RLS) e *Column Level Security* (CLS) no banco. O vendedor "João" faz `SELECT * FROM vendas` e o banco automaticamente adiciona `WHERE vendedor = 'João'` de forma invisível.

**⚖️ Consequências:**
- ✅ Uma única tabela serve dezenas de perfis com segurança transparente.
- ❌ Overhead de performance: o banco avalia as políticas de segurança em cada query.

**💻 Exemplo:**
```sql
-- PostgreSQL — Row Level Security
ALTER TABLE vendas ENABLE ROW LEVEL SECURITY;
CREATE POLICY vendedor_ve_so_dele ON vendas USING (vendedor = current_user);
GRANT SELECT(id, valor, data) ON vendas TO vendedor_joao;  -- Column Level
```

---

### 6.4 Fine-Grained Accessor for Resources
📄 `padroes/07-data-security/fine-grained-accessor-resources.md`

**🔴 Problema:** O script de telemetria usa uma credencial genérica que tem acesso a TODOS os buckets S3 da empresa. Se o script for comprometido, tudo está exposto.

**🟢 Solução:** Aplicar o princípio do *Least Privilege* via IAM Policies. Criar uma identidade (Role) específica para o script que só pode fazer `s3:GetObject` na pasta `s3://telemetria/`. DELETE, PUT e qualquer outro bucket estão proibidos.

**⚖️ Consequências:**
- ✅ Defesa máxima contra comprometimento de credenciais.
- ❌ Pesadelo operacional: gerenciar centenas de políticas IAM granulares é complexo.

**💻 Exemplo:**
```json
{
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:Get*", "s3:List*"],
    "Resource": ["arn:aws:s3:::bucket-telemetria/*"]
  }]
}
```

---

### 6.5 Encryptor (Criptografador)
📄 `padroes/07-data-security/encryptor.md`

**🔴 Problema:** E se o administrador da nuvem baixar o disco rígido fisicamente? E se hackers capturarem pacotes na rede?

**🟢 Solução:** Criptografia em duas frentes: **At-Rest** (dados no disco são embaralhados com chaves KMS — só quem tem a chave lê) e **In-Transit** (TLS obrigatório em toda comunicação de rede).

**⚖️ Consequências:**
- ✅ Dados vazados são lixo matemático sem a chave.
- ❌ Overhead de CPU para criptografar/descriptografar a cada operação.
- ❌ Perder a chave KMS = perder os dados para sempre.

**💻 Exemplo:**
```hcl
# Terraform — Forçar criptografia KMS no S3
resource "aws_s3_bucket_server_side_encryption_configuration" "dados" {
  bucket = aws_s3_bucket.dados.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
      kms_master_key_id = module.kms.key_arn
    }
  }
}
```

---

### 6.6 Anonymizer (Anonimizador)
📄 `padroes/07-data-security/anonymizer.md`

**🔴 Problema:** O time de Data Science precisa de dados para treinar modelos, mas fornecer a tabela real com nomes e CPFs viola a LGPD.

**🟢 Solução:** Destruir irreversivelmente a identidade: trocar nomes por NULL, adicionar ruído aleatório na idade (±2 anos), agrupar em faixas ("20-30 anos"). O dado mantém valor estatístico mas não identifica ninguém.

**⚖️ Consequências:**
- ✅ Dados anônimos estão fora do escopo da LGPD por definição legal.
- ❌ Perda de utilidade: não dá para mandar e-mail para um hash.
- ❌ Risco de re-identificação por triangulação (idade + cidade + data de nascimento).

**💻 Exemplo:**
```python
# PySpark — Anonimização com ruído
from pyspark.sql.functions import rand
df_anonimo = df.withColumn("idade", (col("idade") + (rand() * 4 - 2).cast("int"))) \
               .drop("nome", "cpf", "email")
```

---

### 6.7 Pseudo-Anonymizer (Pseudo-Anonimizador)
📄 `padroes/07-data-security/pseudo-anonymizer.md`

**🔴 Problema:** Anonimizar completamente destrói a capacidade de cruzar dados do mesmo cliente ao longo do tempo. Precisamos de um "disfarce reversível".

**🟢 Solução:** Tokenização: substituir `joao@email.com` por um token `A93Bx` usando criptografia determinística. Todas as tabelas usam `A93Bx`. Quando necessário (ex: investigação de fraude), um administrador com a chave reverte o token para o e-mail real.

**⚖️ Consequências:**
- ✅ Mantém correlações analíticas (JOINs funcionam com tokens).
- ❌ Dados pseudo-anônimos AINDA são dados pessoais perante a lei (podem ser revertidos).
- ❌ O cofre de chaves vira ponto único de falha (SPOF).

**💻 Exemplo:**
```python
# Pseudo-anonimização com criptografia determinística
from cryptography.fernet import Fernet
cipher = Fernet(chave_sistema)
token = cipher.encrypt(b"joao@email.com")  # A93Bx...
original = cipher.decrypt(token)  # joao@email.com (só admin com a chave)
```

---

### 6.8 Secrets Pointer (Apontador de Segredos)
📄 `padroes/07-data-security/secrets-pointer.md`

**🔴 Problema:** A senha do banco está hard-coded no código Python: `pwd = "senha123"`. O código é versionado no Git e a senha vaza para 50 desenvolvedores.

**🟢 Solução:** Armazenar a senha num cofre (AWS Secrets Manager, Azure Key Vault). O código referencia apenas o *ponteiro*: `pwd = os.environ("PROD_DB_PWD")`. O cofre injeta a senha na memória em runtime.

**⚖️ Consequências:**
- ✅ Senhas nunca aparecem no código-fonte nem no Git.
- ✅ Rotação de senhas sem alterar código nem fazer redeploy.
- ❌ Custo: cada chamada ao cofre é cobrada; 500 nós Spark = 500 chamadas.

**💻 Exemplo:**
```python
# Python — Buscando senha do cofre AWS em runtime
import boto3, json
cliente = boto3.client('secretsmanager')
resposta = cliente.get_secret_value(SecretId="producao/banco/main")
senha = json.loads(resposta['SecretString'])['password']
```

---

### 6.9 Secretless Connector (Conector Sem Segredos)
📄 `padroes/07-data-security/secretless-connector.md`

**🔴 Problema:** Mesmo no cofre, a senha ainda existe. Alguém pode fazer `print(senha)` maliciosamente.

**🟢 Solução:** Eliminar senhas completamente. A máquina (EC2, Lambda) se autentica via identidade IAM. O banco confia no IAM e emite um token efêmero de 15 minutos. Não existe senha para vazar.

**⚖️ Consequências:**
- ✅ Segurança máxima: nenhuma senha existe em nenhum momento.
- ❌ Nem todos os bancos/drivers suportam autenticação IAM.
- ❌ Autenticação entre nuvens diferentes (AWS → Azure) é extremamente complexa.

**💻 Exemplo:**
```python
# AWS — Token efêmero IAM para RDS
import boto3
token = boto3.client('rds').generate_db_auth_token(
    DBHostname="banco.aws.com", Port=5432, DBUsername="user_iam")
db.connect(user="user_iam", password=token)  # Token expira em 15 min
```

---

## 7. Armazenamento de Dados

*Como organizar e salvar dados para otimizar custo e performance.*

---

### 7.1 Object Storage (Armazenamento de Objetos)
📄 `padroes/08-data-storage/object-storage.md`

**🔴 Problema:** Terabytes de JSONs não estruturados por dia. Um banco SQL relacional não aguenta ingerir isso nem em termos de formato nem de volume.

**🟢 Solução:** Jogar os arquivos crus direto no Object Storage (S3, GCS, Azure Blob). Ele aceita qualquer formato sem validação prévia (*Schema-on-Read*). A inteligência de leitura fica por conta do motor que consulta depois.

**⚖️ Consequências:**
- ✅ O armazenamento mais barato e escalável que existe.
- ❌ Sem motor de computação embutido: você não pode fazer `UPDATE` diretamente.
- ❌ Sem governança, vira um pântano de dados (*Data Swamp*).

**💻 Exemplo:**
```python
# Spark — Gravação agnóstica no S3
df.write.format("json").save("s3a://data-lake/raw/vendas/")
```

---

### 7.2 External Table (Tabela Externa)
📄 `padroes/08-data-storage/external-table.md`

**🔴 Problema:** Analistas que só sabem SQL precisam consultar arquivos Parquet soltos no S3. Copiar tudo para dentro do DW é caro.

**🟢 Solução:** Criar uma "casca" no banco (tabela externa) que aponta para os arquivos no S3. Quando o analista faz `SELECT`, o banco vai até o S3, lê os arquivos e retorna o resultado. Nenhum dado é copiado.

**⚖️ Consequências:**
- ✅ Sem duplicação de dados; sem vendor lock-in.
- ❌ Performance inferior a tabelas nativas (sem índices, sem otimizações internas).
- ❌ `DROP TABLE` só apaga a casca; os arquivos no S3 ficam intactos (vantagem e desvantagem).

**💻 Exemplo:**
```sql
-- Athena/Hive — Tabela externa apontando para S3
CREATE EXTERNAL TABLE vendas (id INT, valor DECIMAL)
STORED AS PARQUET LOCATION 's3://lago/vendas/';
```

---

### 7.3 Internal Table (Tabela Interna / Gerenciada)
📄 `padroes/08-data-storage/internal-table.md`

**🔴 Problema:** Tabelas externas são lentas para consultas analíticas pesadas. O banco não consegue otimizar o que não controla.

**🟢 Solução:** Internalizar os dados: o banco copia os arquivos para seu armazenamento proprietário, reformata, cria índices e micro-partições. Manutenção (VACUUM, OPTIMIZE) é automática.

**⚖️ Consequências:**
- ✅ Performance insana para consultas analíticas.
- ❌ Vendor lock-in: os dados estão presos no formato proprietário.
- ❌ `DROP TABLE` destrói os dados físicos permanentemente.

**💻 Exemplo:**
```sql
-- Delta Lake — Tabela gerenciada
CREATE TABLE vendas_internas USING DELTA AS SELECT * FROM dados_fonte;
```

---

### 7.4 Vertical Partitioner — Armazenamento
📄 `padroes/08-data-storage/vertical-partitioner-storage.md`

**🔴 Problema:** A tabela tem 100 colunas, mas os analistas só usam 3. Ler 100 colunas para usar 3 desperdiça 97% do I/O.

**🟢 Solução:** Usar formatos colunares (Parquet, ORC). Eles armazenam cada coluna separadamente no disco. Quando o analista pede `SELECT nome, data`, o motor lê fisicamente apenas essas 2 colunas, ignorando as outras 98.

**⚖️ Consequências:**
- ✅ Economia colossal em sistemas que cobram por GB varrido (Athena, BigQuery).
- ❌ `SELECT *` é mais lento que em formatos orientados a linha (o motor precisa reconstruir as linhas).

**💻 Exemplo:**
```sql
-- BigQuery — Projeção estrita (boa prática)
SELECT nome, data FROM vendas;  -- Lê 2 colunas (barato)
-- NUNCA: SELECT * FROM vendas;  -- Lê todas as colunas (caro)
```

---

### 7.5 Horizontal Partitioner (Particionador Horizontal)
📄 `padroes/08-data-storage/horizontal-partitioner.md`

**🔴 Problema:** O Data Lake tem 50 TB de vendas desde 2010. A diretoria quer apenas as vendas de ontem, mas o banco precisa ler os 50 TB para encontrá-las.

**🟢 Solução:** Particionar os dados por uma chave (ex: data). O Spark grava em pastas: `vendas/data=2024-01-15/`, `vendas/data=2024-01-16/`. Ao filtrar `WHERE data = '2024-01-15'`, o motor abre apenas essa pasta, pulando 99% dos dados.

**⚖️ Consequências:**
- ✅ Leitura até 1000x mais rápida quando o filtro usa a chave de partição.
- ❌ Particionar demais (ex: por minuto) cria milhões de micro-pastas (*Small Files Problem*).
- ❌ Consultas que NÃO filtram pela chave de partição perdem todo o benefício (Full Scan).

**💻 Exemplo:**
```python
# Spark — Particionamento por data
df.write.partitionBy("ano", "mes").parquet("s3://lago/vendas/")
# Gera: s3://lago/vendas/ano=2024/mes=01/part-001.parquet
```

---

### 7.6 Bucket (Baldeamento)
📄 `padroes/08-data-storage/bucket.md`

**🔴 Problema:** Não dá para particionar por `cliente_id` (50 milhões de valores únicos = 50 milhões de pastas). Mas precisamos otimizar JOINs por `cliente_id`.

**🟢 Solução:** Distribuir os IDs em N baldes fixos (ex: 200) usando uma função hash. O `cliente_123` sempre cai no balde 47. Se ambas as tabelas usarem 200 baldes, o JOIN acontece localmente em cada balde sem Shuffle.

**⚖️ Consequências:**
- ✅ JOINs distribuídos sem Shuffle: performance dramática.
- ❌ Número de baldes é imutável: mudar exige reescrever toda a tabela.
- ❌ Ambas as tabelas devem ter o mesmo número de baldes para o benefício funcionar.

**💻 Exemplo:**
```sql
-- Hive/Spark SQL — Tabela com bucketing
CREATE TABLE pedidos (cliente_id INT, valor DECIMAL)
CLUSTERED BY (cliente_id) INTO 200 BUCKETS STORED AS PARQUET;
```

---

### 7.7 Logical View (Visão Lógica)
📄 `padroes/08-data-storage/logical-view.md`

**🔴 Problema:** A métrica "Vendas Líquidas" cruza 5 tabelas. Cada analista escreve o cálculo diferente, gerando números inconsistentes.

**🟢 Solução:** Encapsular o SQL complexo numa VIEW. A View não armazena dados — é apenas a definição salva da regra de negócio. Todos os analistas consultam a mesma View e obtêm o mesmo resultado.

**⚖️ Consequências:**
- ✅ Fonte única da verdade; zero duplicação de dados.
- ❌ Recalcula do zero a cada SELECT (sem cache). 50 consultas = 50 recálculos.
- ❌ Views empilhadas (view sobre view) criam labirintos de performance.

**💻 Exemplo:**
```sql
CREATE VIEW vendas_liquidas AS
SELECT v.id, v.valor - d.desconto - i.imposto AS liquido
FROM vendas v JOIN descontos d ON v.id = d.venda_id
JOIN impostos i ON v.id = i.venda_id;
```

---

### 7.8 Materialized View (Visão Materializada)
📄 `padroes/08-data-storage/materialized-view.md`

**🔴 Problema:** 50 diretores abrem o mesmo dashboard que executa um JOIN de 10 bilhões de linhas. São 50 execuções idênticas do mesmo cálculo pesado.

**🟢 Solução:** Materializar o resultado: o banco executa o cálculo uma vez e salva o resultado no disco como uma "fotografia". As 50 consultas batem na foto e retornam em milissegundos. A foto é atualizada periodicamente (REFRESH).

**⚖️ Consequências:**
- ✅ Resposta instantânea para queries repetitivas de BI.
- ❌ Dados podem estar atrasados entre um REFRESH e outro.
- ❌ O REFRESH pode ser pesado se não suportar modo incremental.

**💻 Exemplo:**
```sql
-- PostgreSQL/Redshift — Visão Materializada
CREATE MATERIALIZED VIEW mv_vendas_cidade AS
SELECT cidade, SUM(valor) FROM vendas GROUP BY cidade;

REFRESH MATERIALIZED VIEW mv_vendas_cidade;  -- Atualiza a fotografia
```

---

## 8. Manutenção e Qualidade

*Manter a saúde, a limpeza e a visibilidade do ecossistema de dados.*

---

### 8.1 Quality Checker (Verificador de Qualidade)
📄 `padroes/09-data-maintenance/quality-checker.md`

**🔴 Problema:** Um bug no app fez todas as idades virem `-1`. O pipeline copiou os `-1` perfeitamente sem erro. Meses depois, o modelo de ML faliu.

**🟢 Solução:** Inserir asserções de negócio no pipeline *antes* da carga final: "idade > 0", "email não é nulo", "id é único". Se qualquer regra falhar, o job para (Fail-Fast) e não polui a base.

**⚖️ Consequências:**
- ✅ Impede que lixo entre na base de produção (anti-GIGO).
- ❌ Regras muito rígidas causam falsos positivos (ex: `-1` pode ser "não informado" intencional).
- ❌ Acúmulo de centenas de regras torna o pipeline mais lento que o processamento em si.

**💻 Exemplo:**
```python
# Verificação Fail-Fast antes da carga
erros = df.filter("idade < 0").count()
if erros > 0:
    raise ValueError(f"BLOQUEADO: {erros} idades negativas detectadas!")
df.write.save("base_producao")
```

---

### 8.2 Continuous Monitor (Monitor Contínuo)
📄 `padroes/09-data-maintenance/continuous-monitor.md`

**🔴 Problema:** O Quality Checker bloqueou o pipeline num domingo de feriado porque a receita caiu de 10k para 5k — valor normal para feriado, não um erro.

**🟢 Solução:** Em vez de bloquear, apenas monitorar. O pipeline sempre roda e grava os dados. Paralelamente, um monitor compila estatísticas (média, contagem de nulos, desvios) e alerta a equipe no Slack se algo fugir do padrão, sem parar o fluxo.

**⚖️ Consequências:**
- ✅ Pipeline nunca para; SLAs comerciais são respeitados.
- ❌ Dados ruins já estão na base quando o alerta chega — exige backfilling manual.

**💻 Exemplo:**
```python
# Monitor paralelo que não bloqueia a pipeline
nulos = df.filter(col("email").isNull()).count()
taxa = (nulos / df.count()) * 100
enviar_metrica("taxa_nulos_email", taxa)  # Envia para Datadog/Grafana
df.write.save("base_producao")  # Sempre grava, não importa a taxa
```

---

### 8.3 Isolated Monitor (Monitor Isolado)
📄 `padroes/09-data-maintenance/isolated-monitor.md`

**🔴 Problema:** Adicionar novas regras de qualidade exige abrir o código crítico do pipeline de pagamentos, aprovar PRs arriscados e fazer deploy.

**🟢 Solução:** Desacoplar completamente: o pipeline apenas grava os dados. Uma DAG separada (operada pelo time de Governança) roda depois, conecta-se à tabela já gravada e executa os testes sem tocar no código do pipeline.

**⚖️ Consequências:**
- ✅ Time de Governança tem autonomia total; zero risco ao pipeline.
- ❌ Testes rodam depois da gravação — problemas são detectados tardiamente.

**💻 Exemplo:**
```sql
-- dbt test rodando isolado do pipeline de ingestão
SELECT id_produto, COUNT(*) as duplicatas FROM catalogo
GROUP BY id_produto HAVING COUNT(*) > 1;
-- Se retornar linhas, o teste falha e alerta a equipe
```

---

### 8.4 Metadata Purger (Expurgador de Metadados)
📄 `padroes/09-data-maintenance/metadata-purger.md`

**🔴 Problema:** Um simples `SELECT count(*)` no Iceberg leva 5 minutos. O motor está varrendo gigabytes de logs de versões, snapshots antigos e caminhos de arquivos mortos.

**🟢 Solução:** Programar limpeza periódica dos metadados: `expire_snapshots` remove versões antigas, `remove_orphan_files` limpa arquivos órfãos que o catálogo não referencia mais.

**⚖️ Consequências:**
- ✅ Recupera a performance e reduz custo de armazenamento de metadados.
- ❌ Time-Travel é destruído: não será mais possível consultar o estado da tabela no Natal passado.

**💻 Exemplo:**
```sql
-- Iceberg — Limpeza de snapshots e arquivos órfãos
CALL catalog.system.expire_snapshots('tabela', TIMESTAMP '2024-01-01 00:00:00');
CALL catalog.system.remove_orphan_files(table => 'tabela');
```

---

### 8.5 Structural Compactor (Compactador Estrutural)
📄 `padroes/09-data-maintenance/structural-compactor.md`

**🔴 Problema:** Milhares de DELETEs LGPD fragmentaram a tabela em milhões de arquivos minúsculos de 5 MB cada.

**🟢 Solução:** O comando `OPTIMIZE` lê as partições fragmentadas, unifica os micro-arquivos em blocos ideais (500 MB) e aplica Z-Ordering para alinhar dados similares fisicamente.

**⚖️ Consequências:**
- ✅ Regenera a performance de leitura danificada por fragmentação.
- ❌ Custo alto de computação: compactar TBs exige cluster poderoso.
- ❌ Deve ser aplicado apenas nas partições que mudaram, não na tabela inteira.

**💻 Exemplo:**
```sql
-- Delta Lake — OPTIMIZE com Z-Order
OPTIMIZE vendas WHERE ano = 2024 ZORDER BY (vendedor_id);
```

---

### 8.6 Garbage Collector (Coletor de Lixo)
📄 `padroes/09-data-maintenance/garbage-collector.md`

**🔴 Problema:** 15 anos de dados mortos no S3 "Standard" custam US$ 40 mil/mês. O regulatório só exige 5 anos.

**🟢 Solução:** Configurar *Lifecycle Policies* direto no S3. Após 30 dias, mover para Glacier (centavos). Após 365 dias, deletar permanentemente. Tudo automático, sem código.

**⚖️ Consequências:**
- ✅ Economia massiva e conformidade automática de retenção.
- ❌ Regra mal configurada pode apagar a empresa inteira em 1 dia.
- ❌ Se o catálogo do banco não for notificado, queries vão falhar com "arquivo não encontrado".

**💻 Exemplo:**
```hcl
# Terraform — Lifecycle Policy S3
resource "aws_s3_bucket_lifecycle_configuration" "limpeza" {
  bucket = aws_s3_bucket.logs.id
  rule {
    id     = "apagar_apos_1_ano"
    status = "Enabled"
    filter { prefix = "logs/" }
    expiration { days = 365 }
  }
}
```

---

### 8.7 Lineage Tracker (Rastreador de Linhagem)
📄 `padroes/09-data-maintenance/lineage-tracker.md`

**🔴 Problema:** O lucro no painel caiu. Ele vem da tabela Ouro, que vem da Prata, que lê de 5 fontes Bronze mantidas por 3 times. Ninguém sabe onde o erro começou.

**🟢 Solução:** Ferramentas como OpenLineage capturam automaticamente a cada execução: "Job X leu Tabela A e gravou Tabela B". Um servidor central (Marquez, Atlas) constrói o grafo de dependências. Basta consultar "quem alimenta o painel Ouro?" para ver toda a árvore.

**⚖️ Consequências:**
- ✅ Root Cause Analysis em minutos em vez de dias.
- ✅ Impact Analysis: antes de deletar uma tabela, veja quem depende dela.
- ❌ Scripts legados fora do ecossistema de rastreamento criam "pontos cegos" no grafo.

**💻 Exemplo:**
```yaml
# Airflow — Configuração OpenLineage via variáveis de ambiente
OPENLINEAGE_URL: "http://marquez:5000"
OPENLINEAGE_NAMESPACE: "producao_brasil"
# O Airflow emite eventos de linhagem automaticamente a cada task SQL
```

---

### 8.8 Data Reprofiler (Re-Perfilador de Dados)
📄 `padroes/09-data-maintenance/data-reprofiler.md`

**🔴 Problema:** Corrigimos a lógica de cálculo de sessões em janeiro de 2024. Mas 12 meses anteriores usam a lógica antiga. O CFO quer comparar 2023 vs 2024 com a mesma régua.

**🟢 Solução:** Backfill massivo: disparar o pipeline corrigido retroativamente sobre todas as partições históricas de 2023 usando `airflow dags backfill`. Usar Concurrent Runner para paralelizar e acelerar.

**⚖️ Consequências:**
- ✅ Unifica a verdade analítica; elimina a mancha "daqui pra trás era diferente".
- ❌ Custo computacional brutal: reprocessar 1 ano pode custar mais que 1 mês inteiro de operação normal.
- ❌ Efeito dominó: reprocessar a camada Prata obriga a reprocessar todas as camadas Ouro dependentes.

**💻 Exemplo:**
```bash
# Airflow — Backfill de 1 ano inteiro
airflow dags backfill -s 2023-01-01 -e 2023-12-31 --reset-dagruns pipeline_sessoes
```

---

*Fim do guia de estudo. Para detalhes completos de cada padrão, consulte os arquivos `.md` individuais na pasta `padroes/`.*
