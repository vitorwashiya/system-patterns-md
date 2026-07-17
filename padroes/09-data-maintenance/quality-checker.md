# Padrão: Verificador de Qualidade (Quality Checker)

## 1. Resumo (O que é?)
O padrão Verificador de Qualidade atua como um inspetor da linha de montagem inserido *dentro* do pipeline de dados. Ele testa se os dados atendem às expectativas mínimas de negócio e estrutura. Se os dados passarem, o fluxo continua. Se falharem, ele barra o pipeline, disparando alarmes (Fail-Fast).

## 2. O Problema
* O pipeline de ingestão da empresa nunca quebra. Ele ativamente pega os dados e salva na tabela final.
* Um dia, o desenvolvedor do aplicativo móvel cometeu um erro e a Idade de todos os usuários novos passou a vir com o valor `-1`. O pipeline de ingestão copiou o valor `-1` perfeitamente, não ocorreu nenhum "Erro de Sistema".
* Meses depois, o modelo de Machine Learning faliu porque previu coisas bizarras usando as milhares de idades negativas da base.

## 3. A Solução
Não basta checar infraestrutura. Você introduz o Verificador de Qualidade no meio do código (usualmente após a transformação, antes do carregamento final). Essa verificação se compõe de regras explícitas de negócio. O padrão intercepta o dataframe (ou tabela estagiária) e afirma: "Afirmo que a coluna `Idade` não contém nulos; Afirmo que a `Idade` é sempre `> 0`; Afirmo que `ID` é único". Ferramentas modernas executam isso usando Asserções (*Assertions*) baseadas em métricas. Se a métrica violar o esperado, o job "crasha" (quebra intencionalmente), não deixando o dado poluído atingir o Data Lake de Produção.

## 4. Consequências e Trade-offs
* **Vantagens:** O pilar supremo contra lixo (GIGO - Garbage In, Garbage Out). Evita a poluição silenciosa da base histórica que exige meses de engenharia investigativa para ser purgada depois.
* **Desvantagens/Atenção:** 
  * **O Aumento Constante de Complexidade:** Cada vez que algo dá errado, a equipe adiciona um novo Verificador. Em pouco tempo, o script terá 500 verificações demorando mais para executar as checagens do que para realmente processar o dado da empresa.
  * **Rigidez e Falso Positivos:** Às vezes o dado `-1` é um evento intencional que a origem começou a mandar para dizer "Não informado". Mas como sua pipeline impõe a regra cega `> 0`, ela vai quebrar todos os dias bloqueando o ecossistema até alguém refatorar a verificação.

## 5. Exemplo de Aplicação Prática
Utilizando `Great Expectations` acoplado ao Airflow. Antes da DAG enviar os dados do RH para a tabela Ouro de "Pagamentos", o `Great Expectations` roda um perfilamento atestando que a coluna "Salário" não ultrapassou 10 milhões para nenhum registro (impedindo fraudes ou erros de conversão de ponto). Tudo certo? Libera o commit final.

## 6. Exemplo Simples de Código
```python
# Verificação de qualidade usando Soda Core ou PySpark simples
def verificar_qualidade_dados(df):
    total_linhas = df.count()
    if total_linhas == 0:
        raise ValueError("Qualidade Falhou: O arquivo não tem linhas!")
    
    erros_idade = df.filter("idade < 0").count()
    if erros_idade > 0:
        # Quebra a pipeline e bloqueia antes de poluir a base (Fail-Fast)
        raise ValueError(f"Qualidade Falhou: {erros_idade} idades são menores que zero!")
```

## 7. Padrões Relacionados ou Nomes Similares
Uma fundação primária em ecossistemas conhecidos como *Data Observability*. Possui variações mais soltas como o *Continuous Monitor* que apenas alerta sem parar as máquinas.
