# Padrão: Sequenciador Local (Local Sequencer)

## 1. Resumo (O que é?)
O Sequenciador Local é um padrão para orquestrar tarefas localmente, ou seja, dentro do mesmo pipeline ou trabalho de processamento de dados. Ele desmembra uma lógica enorme em etapas menores, garantindo que sejam executadas de forma sequencial (uma após a outra).

## 2. O Problema
* Seu job de análise é antigo e o código cresceu demais, possuindo centenas de linhas fazendo várias transformações e leituras ao mesmo tempo.
* Qualquer pequena falha exige que o job recomece inteiro do zero, tornando as jornadas de *debug* longas e a manutenção difícil.

## 3. A Solução
Decompor a lógica gigante em tarefas menores conectadas sequencialmente, seguindo as dependências dos dados (ex: Tarefa B precisa dos dados criados pela Tarefa A, então executa após a A). Essa orquestração linear pode ser feita na Camada de Orquestração (como tasks conectadas no Airflow) ou na Camada de Processamento (declarando passos explícitos no PySpark). O foco é priorizar separação de preocupações (*Separation of Concerns*) e facilidade de *Backfilling*.

## 4. Consequências e Trade-offs
* **Vantagens:** Otimiza muito a manutenção e a legibilidade do código, isolando falhas a pontos de *restart* específicos.
* **Desvantagens/Atenção:** 
  * **Definição de Fronteiras:** Quebrar tarefas demais (excesso de granularidade) afoga o orquestrador e torna a observabilidade poluída. Uma boa regra prática é separar lógicas que têm limites lógicos de "Restart" ou exigem diferentes transações ou poder computacional pesado.
  * **Esforço de Implementação:** Requer configurar dependências rigorosas e pode forçar o aprendizado da linguagem específica (DSL) do Orquestrador de Dados.

## 5. Exemplo de Aplicação Prática
Um fluxo de ingestão possui a verificação da existência de arquivo e o carregamento. Se fosse feito num script Python único, um *timeout* na carga faria a verificação de arquivo ocorrer novamente. Usando o Sequenciador Local via Airflow, a verificação e o carregamento são separados; a carga cai, basta apertar o botão "Restart" apenas na etapa de carregamento.

## 6. Exemplo Simples de Código
```python
# Sequenciamento Local no Apache Airflow
# A notação >> garante que a segunda dependa da primeira explicitamente
sensor_de_prontidao_dos_dados >> carga_dos_dados_na_tabela >> expor_nova_tabela
```

## 7. Padrões Relacionados ou Nomes Similares
Oposto da orquestração isolada (como o *Isolated Sequencer*). Relaciona-se ao *Readiness Marker* e trabalha com os princípios básicos aplicados em orquestração de DAGs.
