# Padrão: Rastreador de Linhagem (Lineage Tracker)

## 1. Resumo (O que é?)
O padrão Rastreador de Linhagem (Lineage Tracker) não interfere no processamento analítico direto da ferramenta, mas atua como um radar espião focado puramente em gerar e desenhar as teias de dependência "Raiz-Filho" entre tabelas, campos e jobs, respondendo à pergunta existencial: "De onde diabos veio este número e para onde ele vai?".

## 2. O Problema
* O Analista de Dados identificou que o lucro no painel "Ouro" despencou. O lucro Ouro vem do Painel "Prata", que lê de 5 fontes "Bronze", preenchidas por dezenas de times.
* Ninguém sabe mais quem escreve naquelas fontes originais ou qual coluna "quebrou" porque o ecossistema é formado por 8 mil visões lógicas e 3 mil jobs do Spark espalhados em vários repositórios. Como debugar?

## 3. A Solução
Acoplar agentes que inspecionam o uso do sistema (Lineage Trackers) em cada ponta do fluxo. Utilizando frameworks como **OpenLineage**, toda vez que o Spark, ou dbt, ou Airflow rodam um código com sucesso, eles emitem um pequeno JSON contendo `{"LeuDe": "Tabela A", "GravouEm": "Tabela B"}` para um servidor central corporativo (ex: Apache Atlas ou Marquez). Esse serviço aglutina tudo e plota um mapa visual gigante em árvore (grafo). Dessa forma, qualquer impacto (Análise de Impacto) pode ser visto visualmente (Data Lineage), e se algo quebrar na raiz, o sistema consegue notificar todos os 5 mil painéis filhos sobre o alerta.

## 4. Consequências e Trade-offs
* **Vantagens:** Resolve a "Caixa Preta" de ecossistemas Data Mesh complexos. Acelera investigações de Root Cause Analysis (Causa Raiz) e simplifica os controles para conformidade.
* **Desvantagens/Atenção:** 
  * **Cego à Transparência de SQL Oculto:** Muitas vezes o rastreamento nativo não consegue ler lógicas matemáticas complexas inseridas em consultas obscuras. Por exemplo, se a coluna dependeu da função `CONCAT` e uma tabela temporária bizarra for criada dentro de uma *UDF* solta, os extratores pularão a coluna, gerando pontes soltas (ilhas).
  * **Requerimento Forte de Ferramental Homogêneo:** O rastreamento funciona perfeitamente quando você usa orquestradores que suportam OpenLineage. Se alguém roda um script Perl legadão do servidor embaixo da mesa que altera os arquivos CSVs em segredo, essa tabela fantasma destrói a confiança do painel oficial.

## 5. Exemplo de Aplicação Prática
Para controlar as atualizações de tabelas da Black Friday, um analista quer deletar a "Tabela Desconto Antiga". Ele abre o *Apache Atlas*, digita o nome e o sistema mostra um diagrama em árvore que aponta que, se ele deletar, derrubará imediatamente as Views de Faturamento do Setor B. Ele aborta e avisa o Setor B primeiro (Impact Analysis).

## 6. Exemplo Simples de Código
```yaml
# A configuração muitas vezes não requer código da pipeline, mas 
# configurações no momento da Injeção/Deploy. (Exemplo em YAML)
# Configuração de OpenLineage enviada pelas variáveis de ambiente do Airflow:
OPENLINEAGE_URL: "http://meu-servidor-marquez:5000"
OPENLINEAGE_NAMESPACE: "airflow_producao_brasil"
# O Airflow fará a coleta nos bastidores sempre que qualquer task SQL rodar.
```

## 7. Padrões Relacionados ou Nomes Similares
Base centralizadora da governança em nuvens modernas, costuma andar em conluio com as disciplinas cobertas pelo *Continuous Monitor*.
