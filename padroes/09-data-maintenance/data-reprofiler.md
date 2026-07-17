# Padrão: Re-Perfilador de Dados (Data Reprofiler)

## 1. Resumo (O que é?)
O padrão Re-Perfilador de Dados (*Data Reprofiler* / Backfilling Massivo) lida estruturalmente com as dores da regressão de negócio: ele atua orquestrando a atualização (ou inserção retroativa) em bloco massivo de um conjunto de regras analíticas recém-aprovadas, aplicando-as e propagando a reescrita em meses, anos ou séculos de dados do passado que possuíam os padrões defasados.

## 2. O Problema
* O time consertou a lógica de "Sessões de Clientes" (*Incremental Sessionizer*) em Janeiro de 2024. A regra nova é perfeita.
* Mas o sistema ainda armazena de Janeiro de 2023 até Janeiro de 2024 com a lógica e métricas antigas. O CFO quer um painel comparando as sessões de 2024 com 2023 sob a mesma nova ótica e métrica idêntica.
* Tentar reescrever os arquivos históricos de 2023 ativando o pipeline de forma manual, um por um, no orquestrador é humanamente inviável (além de bloquear tarefas que deveriam rodar para 2024).

## 3. A Solução
Utilizar um Re-Perfilador Estruturado: O orquestrador de dados é configurado para invocar (Backfill) processos de maneira agressiva e limitada com *Concurrent Runner* ou *Single Runner* (se tiver dependência cronológica estrita). Mas diferente da produção, o script do Reprofiler varre os repositórios particionados de forma cega. 
A estratégia pode envolver: 
1. Mover toda a tabela histórica de anos para um "Clone de Reescrita". 
2. Invocar uma carga de "Atualização de Regra" sobre esse clone maciço utilizando capacidades computacionais extraordinárias. 
3. Quando finalizado, inverter os apontadores (troca de visão), desativando o histórico velho e apontando o Lake principal para a nova massa corrigida.

## 4. Consequências e Trade-offs
* **Vantagens:** Unifica a verdade corporativa analítica; permite a evolução constante de lógicas sem a eterna mancha do "aqui o dado era X, a partir de hoje é Y".
* **Desvantagens/Atenção:** 
  * **Violência Operacional:** Rodar novamente 3 anos de um banco de dados consumirá mais força, recursos financeiros e latência no banco do que todo o uso mensal habitual somado, o que pode exaurir orçamentos do provedor Cloud sem aprovação corporativa.
  * **Efeitos Colaterais Impensáveis (Butterfly Effect):** Se você corrige a Base Prata de 2023, todas as centenas de Bases Ouro ligadas (veja *Lineage Tracker*) sofrerão impactos. Elas também devem ser Re-Perfiladas após a base Prata, gerando um efeito dominó que pode atrasar entregas de negócio primordiais.

## 5. Exemplo de Aplicação Prática
Para consertar uma lógica tributária defasada (regra que mudou de 5% para 7% retroativamente no ano fiscal), a equipe escreve o Re-Perfilador usando um cluster Spark 5x mais poderoso do que usam diariamente. Ele ataca a raiz do problema varrendo o banco retroativamente, roda as transações velhas e cria uma nova visão histórica.

## 6. Exemplo Simples de Código
```bash
# Via Airflow, as ferramentas são capazes de comandar dezenas de Reprofilers via linha de comando 
# (Clear And Run / Backfill) 
airflow dags backfill \
    -s 2023-01-01 -e 2023-12-31 \
    --reset-dagruns \
    pipeline_tributario_calculo
```

## 7. Padrões Relacionados ou Nomes Similares
Exige maestria no uso e controle paralelo dos padrões *Single Runner* ou *Concurrent Runner*. Trabalha com arquivos maciços muitas vezes usando as travas do *In-Place Overwriter*.
