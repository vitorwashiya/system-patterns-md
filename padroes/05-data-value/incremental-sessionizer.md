# Padrão: Sessionizador Incremental (Incremental Sessionizer)

## 1. Resumo (O que é?)
O padrão de Sessionizador Incremental tenta rastrear e organizar uma sequência de eventos independentes em blocos de comportamento coerentes e sequenciais chamados de "Sessões". Este padrão foca na capacidade de criar e fechar essas janelas lógicas gradativamente em processamentos baseados em lote (Batch).

## 2. O Problema
* Arquiteturas em batch normalmente leem pedaços mensais ou diários dos dados. Um usuário pode navegar no seu site (uma sessão) das 23h50 às 00h30. Essa visita cruzará arquivos pertencentes a partições de tempo distintas (ex: dia de hoje e de amanhã).
* Os analistas da equipe não conseguem agrupar a atividade do usuário porque analisar duas partições de arquivos por vez requer muita lógica que muda frequentemente e estressa a orquestração do banco de dados.

## 3. A Solução
Implementar a criação contínua através do Sessionizador Incremental. O padrão precisa de 3 repositórios físicos (armazenamentos):
1. **Dados Brutos de Entrada:** A partição diária com o histórico bruto.
2. **Sessões Concluídas:** O destino que usuários e analistas leem.
3. **Sessões Pendentes (Em Progresso):** Área oculta utilizada estritamente pelo pipeline.

A lógica cruza os dados do dia corrente com o estado da gaveta "Sessões Pendentes" que foi criado nas execuções dos lotes anteriores. Para cada chave (usuário), ela cria uma nova sessão, ou avança a lógica para continuar as pendentes. No final, ela salva os dados consolidados: joga as sessões "fechadas" (usuários que ficaram inativos além de um limite predeterminado) no banco público de sessões finalizadas e repassa as sessões ativas para a área de sessões pendentes para serem cruzadas com o lote do próximo dia.

## 4. Consequências e Trade-offs
* **Vantagens:** Otimiza absurdamente relatórios longos baseados em batch; previne falhas estruturais em jornadas divididas temporalmente.
* **Desvantagens/Atenção:** 
  * **Janelas de Inatividade:** Sessões pendentes ficam travadas no limbo invisível da tabela privada. Se você tentar emitir sessões "parciais" (ongoing) para seu usuário para mitigar a lentidão, eles poderão ler um "insight parcial" que se reverterá (falso sucesso).
  * **Complexidade Extra ao Lidar com Dados Atrasados:** Como sessões são janelas que empurram os tempos adiante, reprocessar um único dia do passado (late data) destruirá todos os resultados dos dias sucessivos, demandando uma carga infinita de backfilling do ponto que falhou até a data atual.

## 5. Exemplo de Aplicação Prática
Para análise da usabilidade de e-commerce, os relatórios são gerados de hora em hora. A extração das 14h encontra cliques da usuária "Ana" e, como ela clicou às 13h55 e parou, salva isso no "Pendentes". Às 15h, o script lê a Ana e percebe que ela não clicou mais. Por passar o limite de 30 min sem inatividade, ele consolida, limpa os pendentes, e oficializa o encerramento no repositório final de visões analíticas.

## 6. Exemplo Simples de Código
```sql
-- Lógica mesclada do orquestrador gerenciando as Sessões
-- 1. Combina a entrada do dia com pendentes de ontem
CREATE TEMP TABLE sessoes_para_classificar AS 
SELECT ... FROM visitas_hoje OUTER JOIN sessoes_pendentes_ontem

-- 2. Limpa finalizadas da memória
INSERT INTO sessoes_publicas SELECT * FROM sessoes_para_classificar WHERE is_final = true;

-- 3. Passa abertas para memória temporária (pendentes) do dia de amanhã processar
INSERT INTO sessoes_pendentes SELECT * FROM sessoes_para_classificar WHERE is_final = false;
```

## 7. Padrões Relacionados ou Nomes Similares
Padrão muito comum na arquitetura *Lambda* ou processamentos pesados focados em *Incremental Loader*.
