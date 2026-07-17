# Padrão: Anonimizador (Anonymizer)

## 1. Resumo (O que é?)
O Anonimizador é um padrão rígido focado em desvincular irreversivelmente e completamente a informação do indivíduo a quem ela originalmente pertence. Seu objetivo final é produzir conjuntos de dados que possam ser compartilhados globalmente sem ferir diretrizes de segurança, substituindo campos por lixo numérico, apagando-os ou embaralhando os dados a um nível irrecuperável.

## 2. O Problema
* O time de Ciência de Dados quer testar um modelo para prever a idade de compra dos clientes cruzando idade com a cesta de produtos.
* Se você fornecer a tabela ao time, você estará distribuindo a idade de todo mundo (expondo pessoas). Se você mascarar a idade (Masking: colocar "XXXX" na idade), o modelo matemático do time de Dados não vai conseguir somar um monte de "X"s.

## 3. A Solução
Use a anonimização. Em vez de simplesmente mascarar, você emprega funções lógicas que destroem o valor original sem destruir a estrutura. A anonimização conta com diversas vertentes, principalmente:
* **Nulling (Anulação):** Remover os nomes dos usuários simplesmente injetando `NULL`.
* **Randomization/Fuzzing (Ruído):** Alterar aleatoriamente números dentro de faixas seguras. (ex: Somar `+3` ou `-1` na Idade de todo mundo aleatoriamente, ou agrupar os clientes como `Faixa dos 20-30 anos`).
* **Hashing Rígido (sem Salting constante):** Transforma o E-mail em um Hash unidirecional.

## 4. Consequências e Trade-offs
* **Vantagens:** O grau máximo de liberação. Dados efetivamente anonimizados fogem ao escopo da LGPD, pois por lei já não referenciam um indivíduo real detectável.
* **Desvantagens/Atenção:** 
  * **Destruição da Utildade (Utility Loss):** Um Hashing de E-mail salva a privacidade, mas destrói completamente qualquer campanha de marketing via E-mail.
  * **Vulnerabilidade de Inferência Computacional:** Uma idade de "32" sozinha é anônima. Mas se a linha tem "Idade: 32", "Cidade: São Caetano do Sul", "Data Nascimento: 15/09/1990", o cientista consegue triangular as informações com bases públicas até re-identificar a pessoa indiretamente. O anonimizador falha sem contexto amplo.

## 5. Exemplo de Aplicação Prática
Para ceder uma base à parceiros externos em um Hackathon, o pipeline extrai a tabela. Troca os Nomes por "M" ou "F" baseados na letra final. Altera idades como `34` para `30-35`, e deleta o IP. Agora a base de 10 mil linhas é matemática pura baseada em comportamento de consumo geral, impossível de voltar ao usuário 10.

## 6. Exemplo Simples de Código
```python
# Anonimizador em PySpark criando "Fuzzing/Ruído"
from pyspark.sql.functions import rand

# Adicionando ruído e apagando a coluna de Nome Original
dados_anonimos = (
    dados
    .withColumn("idade_anonima", col("idade") + (rand() * 4 - 2).cast("int")) # Entre -2 e +2 de ruído
    .drop("nome_real")
)
```

## 7. Padrões Relacionados ou Nomes Similares
Muitas vezes confundido com o *Pseudo-Anonymizer*, onde o retorno à identidade (de-anonimização) é tecnicamente possível.
