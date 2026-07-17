# Padrão: Sessionizador Com Estado (Stateful Sessionizer)

## 1. Resumo (O que é?)
O padrão de Sessionizador Com Estado é construído para dar suporte a aplicações de streaming em tempo real na criação de sessões, sem utilizar processos batch divididos em tarefas pesadas do banco. Utiliza uma base de armazenamento na memória da ferramenta (State Store) que permite gerar sessões na exata medida em que elas evoluem.

## 2. O Problema
* Os seus clientes de negócios amaram que a sua arquitetura gera sessões agrupadas (Incremental Sessionizer), mas odeiam ter que aguardar a hora fechar (e rodar os cálculos diários) para gerar alguma utilidade a partir disso.
* As partes interessadas exigem processamento instantâneo. Mas transformar as janelas do batch diretamente num script sem memória infinita causaria lentidão e exaustão computacional em ferramentas de streaming sem suporte de retenção de estados.

## 3. A Solução
Substitua pipelines isolados de batch pelas arquiteturas com State Store das ferramentas modernas. A lógica em fluxo contínuo cria "Session Windows". Na prática, para um evento novo de um cliente que chega na fila, o fluxo procura na memória rápida pela sessão pré-existente dele. Adiciona o valor atualizado e registra essa sessão in-flight no cache de tolerância a falhas. Uma variável com um temporizador interno expurgará automaticamente essa chave e enviará os registros finais para a tabela pública caso passem (ex) 20 minutos inativos.

## 4. Consequências e Trade-offs
* **Vantagens:** Sessões geradas incrivelmente rápidas. Resolve a latência limitante presente em *Incremental Sessionizer*.
* **Desvantagens/Atenção:** 
  * **Problemas com Escalonamento (Scaling):** Quando a infraestrutura de tempo real decide adicionar nós aos fluxos em resposta a grandes demandas, ela precisa re-distribuir pedaços de memória RAM compartilhadas entre servidores, que gera sobrecarga momentânea e paralisação dos pipelines dinamicamente.
  * **Risco de "Pelo-Menos-Uma-Vez" em Processamento Interno:** A proteção do pipeline envolve checkpoints automáticos do estado na persistência. Se o script morrer entre processar a RAM e persistir o *commit* físico, os resultados retornarão ao passado. Variáveis como `data_atual` ou "NOW()" devem ser evitadas em chaves por não gerarem resultados idempotentes (podem criar duas janelas de sessão distintas para o mesmo clique que sofreu retry sistêmico).

## 5. Exemplo de Aplicação Prática
Para evitar perdas de fraude em um banco, o sistema ouve todos os saques num cartão de crédito nas últimas duas horas. Um State Store acumula cada saque num objeto em memória do Spark. Ocorrendo o terceiro saque sob aquela chave ("João"), ele já tem a inteligência total que se acumulou temporalmente ali sem ir até a infraestrutura do banco de dados relacional e fecha a sessão de detecção, bloqueando o usuário.

## 6. Exemplo Simples de Código
```python
# Session Windows nativo na Apache Flink (PyFlink) / Spark
sessoes = (
    fluxo_visitas
    .key_by(VisitIdSelector())
    # Flink controla sozinho o cache do Estado em memória, definindo uma janela
    # que apaga com inatividade de 10 minutos (with_gap)
    .window(EventTimeSessionWindows.with_gap(Time.minutes(10)))
    .process(MinhaFuncaoConversoraFinal()) 
)
```

## 7. Padrões Relacionados ou Nomes Similares
Implementação fundamental em plataformas analíticas (e.g. Apache Flink, Kafka Streams). Sofre dos problemas de atraso descritos no *Late Data* (Dados Atrasados). Utiliza as abstrações ativas do *Windowed Deduplicator* internamente.
