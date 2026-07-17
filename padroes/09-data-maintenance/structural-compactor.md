# Padrão: Compactador Estrutural (Structural Compactor)

## 1. Resumo (O que é?)
Diferente do Compactador de Ingestão que une arquivos em streaming para otimizar filas, o Compactador Estrutural (*Structural Compactor* ou *OPTIMIZE/Z-Order*) é um padrão de manutenção executado no banco já consolidado. Ele lê partições inteiras e aplica lógicas geométricas de reorganização (Clustering) nas colunas e une milhares de arquivos soltos em arquivos ideais maiores e com metadados ajustados, acelerando brutalmente as leituras futuras.

## 2. O Problema
* O banco rodou dezenas de exclusões de conformidade (LGPD) através de *In-Place Overwriter*. Isso perfurou a tabela, e agora o sistema possui milhões de fragmentos de arquivos de dados que ficaram com apenas 5 Megabytes úteis.
* Ferramentas colunares foram projetadas para ler arquivos gordos de 200MB a 1GB. Ler um milhão de arquivos minúsculos estressa o disco rígido ("Small Files Problem").

## 3. A Solução
Programar um pipeline de rotina noturna puramente dedicado à Compactação Estrutural (Ex: rodando o comando `OPTIMIZE` em Lakehouses). Durante a madrugada, o motor analítico lê as partições que sofreram fragmentação, unifica fisicamente as micro-frações e grava de volta num limite matematicamente ideal (como blocos puros de 500 MB). Adicionalmente, técnicas avançadas como Z-Ordering permitem que ele embaralhe as linhas, alinhando informações semelhantes umas ao lado das outras fisicamente, acelerando saltos de leitura.

## 4. Consequências e Trade-offs
* **Vantagens:** Regenera o desempenho e a saúde física dos bancos de dados, anulando os estragos provocados por streaming constante e modificações fracionadas de I/O.
* **Desvantagens/Atenção:** 
  * **Custo Exorbitante de Computação:** O comando OPTIMIZE exige que a nuvem carregue um volume de Terabytes na RAM, decomprima, reagrupe na CPU e regrave, gerando contas estratosféricas se rodado desnecessariamente todo dia inteiro na tabela inteira. Você só deve compactar as partições que mudaram recentemente.

## 5. Exemplo de Aplicação Prática
Seu Delta Lake salva de hora em hora pequenos lotes de vendas. No domingo, o Airflow executa um PythonOperator. Esse operador manda a instrução `OPTIMIZE` só para as vendas dos últimos 7 dias. Ele unifica 168 arquivos de 2MB em um super arquivo de 336MB. Consultar a última semana agora leva frações de segundo.

## 6. Exemplo Simples de Código
```sql
-- Compactador Estrutural com alinhamento físico (Z-Ordering) Delta Lake
-- Apenas mexe em arquivos referentes ao ano de 2023 
OPTIMIZE eventos_vendas 
WHERE ano = 2023
-- Alinha geograficamente os registros próximos fisicamente
ZORDER BY (id_vendedor); 
```

## 7. Padrões Relacionados ou Nomes Similares
Uma evolução avançada e em repouso do padrão de *Compactor* de ingestão.
