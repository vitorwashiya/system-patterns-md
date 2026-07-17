# Padrão: Decorador de Metadados (Metadata Decorator)

## 1. Resumo (O que é?)
O padrão Decorador de Metadados foca em injetar informações de contexto puramente técnicas na camada de dados, sem misturar esses detalhes com a estrutura lógica de negócio visível pelos usuários finais.

## 2. O Problema
* Seus pipelines de fluxo contínuo evoluem e ganham versões constantemente.
* Para debugar problemas, a equipe de engenharia quer rastrear o contexto de execução (ex: número do lote, versão do código, timestamp da carga) em cada registro gerado, mas os analistas de dados reclamam que essas colunas técnicas apenas sujam o schema e não servem para nada no Business Intelligence (BI).

## 3. A Solução
Você pode alavancar a camada de metadados do seu sistema de armazenamento para guardar essas informações. No Apache Kafka, isso pode ser feito utilizando os "Headers" nativos das mensagens. Em Object Storage (como S3), pode ser feito adicionando "Tags" nos arquivos. Em Bancos de Dados Analíticos onde isso não seja suportado de forma tão isolada, você cria colunas no banco de dados mas usa ferramentas de segurança e controle de acesso (como o padrão *Fine-Grained Accessor*) ou tabelas complementares separadas e não-públicas para ocultá-las do consumidor de negócios.

## 4. Consequências e Trade-offs
* **Vantagens:** Adiciona total transparência técnica e rastreabilidade para os engenheiros, sem sujar o ecossistema e o fluxo dos analistas de negócios.
* **Desvantagens/Atenção:** 
  * **Suporte de Implementação:** Ferramentas e provedores variam drasticamente na forma como gerenciam metadados ocultos. Alguns brokers (como Amazon Kinesis Data Streams mais antigos) sequer suportam cabeçalhos.
  * **Perigo de Escopo:** Você nunca deve usar a camada técnica de metadados para salvar dados lógicos de negócio (como endereço ou CEP), pois a grande maioria das ferramentas de análise e queries ignoram o nível de metadados durante as consultas por padrão.

## 5. Exemplo de Aplicação Prática
O desenvolvedor atualiza a pipeline e ela passa a adicionar o cabeçalho técnico `job_version=v2.0` no momento em que escreve as mensagens de transações em um tópico do Kafka. O consumidor a jusante (um sistema financeiro) lê as mensagens da mesma forma que antes, sem quebrar os contratos, mas a equipe de engenharia consegue pesquisar quem gerou cada mensagem acessando os metadados.

## 6. Exemplo Simples de Código
```python
# Adicionando Metadata Decorator como Kafka Headers no Spark
dados_com_metadados = (
    dados.withColumn(
        'headers', 
        expr("array(struct('job_version' as key, '1.5.0' as value))")
    )
)

dados_com_metadados.write.format('kafka').option('includeHeaders', 'true').save()
```

## 7. Padrões Relacionados ou Nomes Similares
Uma alternativa isolada e não-obstrusiva ao padrão *Wrapper*. Pode usar *Fine-Grained Accessor for Tables* para sua implementação em DWs relacionais.
