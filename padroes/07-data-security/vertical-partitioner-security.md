# Padrão: Particionador Vertical (Vertical Partitioner)

## 1. Resumo (O que é?)
O Particionador Vertical foca em separar fisicamente um mesmo registro em dois (ou mais) pedaços, alocando os atributos que compõem o registro em tabelas ou arquivos diferentes. Quando aplicado à segurança e conformidade (ex: regras de "Direito ao Esquecimento" da LGPD/GDPR), ele separa atributos sensíveis (PII) de atributos transacionais imutáveis.

## 2. O Problema
* Leis de proteção de dados exigem que você delete dados pessoais de um usuário quando solicitado. 
* A tabela original é gigantesca e armazena os dados pessoais em cada linha da transação de acesso. Tentar achar e deletar os dados do "João" lendo e reescrevendo petabytes de arquivos de transação é dolorosamente lento e caro.

## 3. A Solução
Quebre a linha em duas partes durante o momento da ingestão (Particionamento Vertical). Todos os dados mutáveis e sensíveis do João (PII, nomes, e-mail) vão para um armazenamento de *Identidade*, enquanto todos os eventos imutáveis gerados pelo João (Cliques, Dispositivo, Navegador) vão para outro armazenamento de *Eventos* anonimizado, contendo apenas um "ID cego" para fazer o vínculo. Quando a exclusão for solicitada, você exclui apenas 1 linha curta da tabela de *Identidade* e pronto: o ID cego na tabela gigantesca perde seu vínculo humano permanentemente de forma instantânea.

## 4. Consequências e Trade-offs
* **Vantagens:** Otimiza o custo operacional de varredura para exclusões GDPR/LGPD transformando *deletes* massivos em *deletes* pontuais; protege a identidade do usuário por design.
* **Desvantagens/Atenção:** 
  * **Performance e Complexidade de Consulta:** Qualquer leitor de negócios que precisar ver o dado unificado será forçado a realizar um *JOIN* através da rede para reunir a tabela de identidade com a de eventos, aumentando a latência da consulta. (A mitigação seria o padrão de *Dataset Materializer*).
  * **Complexidade Multi-Bancos:** A separação pode envolver salvar a identidade num banco de chave-valor super restrito e salvar o evento em um Data Lake, ampliando os pontos de falha sistêmicos.

## 5. Exemplo de Aplicação Prática
Seu log de telemetria captura de 1.000 cliques/segundo do usuário. Em vez de salvar `IP`, `Email` e `ID` em toda linha de clique, você guarda `ID=10`, `IP=...`, `Email=...` na Tabela de Usuários 1 vez. Os cliques ficam salvos só com o `ID=10`. Quando o usuário pede exclusão, a tabela de usuários deleta a chave `10`. Os bilhões de cliques de telemetria continuam no sistema gerando volume analítico anônimo para os cientistas, mas ninguém descobre de quem era.

## 6. Exemplo Simples de Código
```python
# Particionando verticalmente usando Apache Spark
# Divide a tabela única separando as colunas em 2 datasets independentes
eventos_sem_pii = dados_originais.drop('endereco', 'nome_cliente')
eventos_sem_pii.write.save("s3://logs_brutos/")

somente_pii = dados_originais.select('id_cliente', 'endereco', 'nome_cliente').dropDuplicates()
somente_pii.write.save("s3://dados_pessoais_protegidos/")
```

## 7. Padrões Relacionados ou Nomes Similares
Uma variação voltada à segurança do *Vertical Partitioner* voltado para performance. Opõe-se em custo ao *In-Place Overwriter*.
