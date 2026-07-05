
# Full Loader

## Classificação
- **Tipo:** Criacional
- **Escopo:** Objeto

## Intenção
Extrair e carregar (EL) um conjunto de dados completo a cada execução, substituindo integralmente a base de destino sem depender de rastreamento de estado.

## Também Conhecido Como
Passthrough Jobs, Extract and Load (EL), Full Load.

## Motivação (Contexto)
Integração de tabelas de referência de sistemas externos que sofrem mutações lentas e não possuem atributos delta (ex: data de atualização). Um carregamento completo reescreve todo o conjunto de dados de uma única vez na camada inicial.

## Aplicabilidade
- Use o Full Loader quando:
  - O sistema de origem omite mecanismos de identificação de alterações incrementais.
  - O volume de dados ingerido cresce lentamente e não impacta as janelas de processamento de forma drástica.
  - O destino requer e suporta substituição total (drop-and-insert).

## Estrutura e Participantes
- **Data Provider:** Origem primária que contém a totalidade dos registros do domínio.
- **Passthrough Job:** Processo de ingestão que movimenta os dados sem aplicar transformações lógicas complexas.
- **Data Target:** Sistema de armazenamento ou exposição que recebe a reescrita completa.

## Colaborações
O Passthrough Job requisita o volume total de dados no Data Provider e aciona operações de substituição integral no Data Target, mantendo a versão unificada e atômica do dataset.

## Consequências
- **Prós:**
  - Redução extrema na complexidade lógica, limitando-se unicamente ao papel de Extract-Load puro.
  - Simplicidade primária na mitigação de falhas e reprocessamentos por conta da idempotência inerente.
- **Contras:**
  - Risco proeminente de gargalos e falhas de infraestrutura caso o volume de dados cresça de forma exponencial não esperada (Volume).
  - Inconsistência momentânea de dados originando consultas parciais durante o ciclo longo do drop-and-insert.

## Implementação (Exemplo de Código)
```python
# Extração e carregamento full com Apache Spark e Delta Lake
input_data = spark.read.schema(input_data_schema).json("s3://devices/list")

# A diretiva mode("overwrite") assegura a natureza substitutiva e completa do Full Loader
input_data.write.format("delta").mode("overwrite").save("s3://master/devices")
```

## Padrões Relacionados

* **Proxy:** Essencial para ocultar a substituição física dos dados dos consumidores finais através de abstrações atômicas como Views.
* **Incremental Loader:** Alternativa direta adotada imediatamente quando a latência ou o custo inviabilizam processamentos completos contínuos.

---
