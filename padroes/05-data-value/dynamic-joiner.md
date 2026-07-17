# Padrão: Junção Dinâmica (Dynamic Joiner)

## 1. Resumo (O que é?)
O padrão de Junção Dinâmica (Dynamic Joiner) é focado em combinar dois fluxos contínuos de dados (streams) que mudam ativamente. Ao invés de uma tabela parada e uma se movendo, ambos os lados são dinâmicos e chegam com latências diferentes.

## 2. O Problema
* Você precisa cruzar dados de eventos de usuários com o fluxo de "atualizações de perfis de usuário". Como as mudanças de perfil fluem rapidamente através de um sistema de Change Data Capture (CDC), a base de usuários não é estática.
* Eventos podem chegar antes de suas atualizações correspondentes no outro fluxo (ou vice-versa), resultando em falhas de junção.

## 3. A Solução
Para cruzar fluxos dinâmicos, ferramentas de streaming aplicam o padrão criando buffers baseados em tempo (Time Boundaries). O fluxo mais rápido aguarda o mais lento até um limite de tempo estabelecido. Os registros de ambos os lados são armazenados em memória temporária (State Store) até encontrarem seu "par" ou expirarem. O tempo de expiração é muitas vezes controlado por uma "Garbage Collection (GC) Watermark".

## 4. Consequências e Trade-offs
* **Vantagens:** Lida elegantemente com descompassos temporais (latência de rede ou atraso na emissão) entre duas fontes de dados em tempo real.
* **Desvantagens/Atenção:** 
  * **Trade-off Espaço vs Exatidão:** Um buffer longo consumirá muita memória e atrasará a emissão do dado, mas fará quase todos os "matches". Um buffer curto gasta menos memória, mas perderá junções.
  * **Dados Atrasados (Late Data):** Dados que chegarem depois do tempo estipulado na Watermark serão descartados, gerando perda na junção.

## 5. Exemplo de Aplicação Prática
Sua empresa emite propagandas e rastreia "Impressões" e "Cliques" em fluxos separados. O Dynamic Joiner cruza os dois fluxos usando `ad_id`, tolerando que o fluxo de "Clique" chegue em até 10 minutos após a "Impressão".

## 6. Exemplo Simples de Código
```python
# Junção baseada em tempo usando Spark Structured Streaming
cliques = fluxo_cliques.withWatermark('horario_clique', '10 minutes')
impressoes = fluxo_impressoes.withWatermark('horario_imp', '10 minutes')

# Define a janela de tempo de tolerância diretamente na condição
cliques_cruzados = cliques.join(
    impressoes,
    f"""
    cliques.ad_id = impressoes.ad_id AND
    horario_clique BETWEEN horario_imp AND horario_imp + INTERVAL 10 minutes
    """,
    'left_outer'
)
```

## 7. Padrões Relacionados ou Nomes Similares
Muitas ferramentas se referem a isso como *Time-windowed Join* ou *Temporal Table Joins* (no Apache Flink). Utiliza o mecanismo base do *Late Data Detector*.
