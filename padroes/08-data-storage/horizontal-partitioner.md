# Padrão: Particionador Horizontal (Horizontal Partitioner)

## 1. Resumo (O que é?)
O Particionador Horizontal é o principal mecanismo estrutural do Big Data para lidar com o gigantismo físico. Ele não mexe nas colunas da tabela; em vez disso, ele fatia as *Linhas* da tabela em blocos ou arquivos separados, baseando-se em uma lógica de agrupamento (uma "Chave de Partição" como `Data` ou `Região`), empacotando linhas com atributos iguais em pastas físicas do sistema operacional isoladas.

## 2. O Problema
* Seu Data Lake tem um arquivo único `vendas.csv` com 50 Terabytes.
* A diretoria pediu apenas as vendas de **Ontem**. 
* O banco de dados precisará ler todos os 50 Terabytes, desde o ano de 2010 até hoje, para conseguir encontrar as poucas linhas espalhadas que têm a data de ontem. Isso gastará horas de processamento, e se o serviço cobrar por escaneamento, a conta será absurda.

## 3. A Solução
Quebre o monólito usando o Particionador Horizontal. Durante a gravação dos dados diários, em vez de anexar tudo no mesmo arquivo, você configura a arquitetura (ex: Spark) com `partitionBy('data')`. O sistema construirá uma árvore de pastas físicas virtuais no disco: `vendas/data=2024-01-01/`, `vendas/data=2024-01-02/`. Todos os registros daquela data são atirados nas suas próprias pastas. Quando a diretoria rodar o comando com filtro (Ex: `WHERE data = '2024-01-02'`), o motor do banco de dados intercepta a requisição, pula 99% das pastas e lê fisicamente apenas a pasta de ontem, escaneando apenas os Megabytes de interesse. 

## 4. Consequências e Trade-offs
* **Vantagens:** A espinha dorsal da performance e eficiência de custo no Big Data. Essencial para isolamento de falhas: se a carga de hoje quebrar, não corromperá os arquivos separados do passado.
* **Desvantagens/Atenção:** 
  * **A Maldição dos Arquivos Pequenos (Small Files):** Particionar agressivamente com uma chave muito granular (ex: particionar por `Minuto` em vez de `Dia`) criará milhões de micro-pastas com arquivos de 2 KB. Abrir e fechar pastas gasta mais tempo que ler o dado.
  * **O Desastre da Pesquisa Cruzada:** Se o banco particionou por `Ano`, mas o RH decide fazer uma pesquisa global ignorando o tempo: `WHERE usuario_nome = 'Maria'`, o particionador se torna inútil. O motor será forçado a abrir e vasculhar as pastas de todos os anos de qualquer forma para procurar a Maria (Full Scan). O particionamento só ajuda se você filtrar a coluna da chave!

## 5. Exemplo de Aplicação Prática
Para salvar arquivos IoT no AWS S3, a equipe particiona no formato de árvore: `s3://logs/ano=2024/mes=05/dia=14/`. O AWS Athena varre apenas a pasta do dia 14 e acha os dados em 1 segundo. Se a chave particionada mudasse para "Cidade", ele também acharia os sensores de "Campinas" isolados magicamente em instantes.

## 6. Exemplo Simples de Código
```python
# Spark particionando fisicamente por colunas na hora de escrever os arquivos Parquet
df_cliques.write.partitionBy("ano", "mes").parquet("s3://lago/cliques/")

# Isso gerará pastas nativas como s3://lago/cliques/ano=2023/mes=01/part-001.parquet
```

## 7. Padrões Relacionados ou Nomes Similares
Costuma ser o padrão primário antes da necessidade fina ditada pelo padrão de *Bucket*. Contrapõe o modelo em eixo do *Vertical Partitioner*.
