# Padrão: Idempotência Baseada em Chave (Keyed Idempotency)

## 1. Resumo (O que é?)
A Idempotência Baseada em Chave é um padrão no qual um banco de dados de chave-valor (ou banco baseado em chaves primárias) é utilizado junto com uma estratégia determinística de geração de chaves. O objetivo é assegurar que, independentemente de quantas vezes um mesmo registro seja processado e enviado, ele só existirá de forma única no banco.

## 2. O Problema
* Seu pipeline de streaming agrupa eventos de visita do usuário em uma janela de tempo e grava o resultado final (a "sessão" do usuário) no banco.
* Devido a retentativas automáticas por falhas intermitentes (*retries*), o pipeline reprocessa alguns eventos e os escreve de novo, gerando registros duplicados da mesma sessão, sujando a base de dados.

## 3. A Solução
Para garantir a idempotência sem precisar de uma ferramenta externa de orquestração, você deve construir a chave primária de forma imutável e previsível. Em vez de criar um ID aleatório no momento da gravação, você utiliza atributos intrínsecos do dado que nunca mudam, por exemplo, unindo o ID do Usuário (`user_id`) com o momento exato em que o primeiro evento entrou no sistema (`append_time` ou `ingestion_time`). Quando a retentativa acontecer, o código irá gerar rigorosamente a mesma chave e, portanto, apenas atualizará o registro existente no banco (ou não fará nada), eliminando o risco de duplicatas descontroladas.

## 4. Consequências e Trade-offs
* **Vantagens:** Protege a saída de dados sem exigir lógicas complexas de validação ou bloqueios de transação pesados. Extremamente veloz em bancos NoSQL.
* **Desvantagens/Atenção:** 
  * **Depende do Banco de Dados:** Funciona excepcionalmente bem em bancos NoSQL baseados em chaves (Cassandra, HBase), mas em bancos relacionais exige comandos específicos de DML como `MERGE` (ou `INSERT ON CONFLICT`) para evitar erros de violação de chave primária.
  * **Fonte Mutável:** Se o dado de origem sofreu exclusões ou compactações que removeram o evento mais antigo usado para gerar a chave, ao reiniciar o job, ele gerará uma chave nova e você perderá a garantia de idempotência.

## 5. Exemplo de Aplicação Prática
Um sensor de temperatura envia o valor a cada minuto. O job compila a média de 10 minutos. Em vez de salvar a média com um "ID UUID aleatório gerado na hora", ele salva usando a chave `sensor123_10h00`. Se o job cair e reprocessar esse mesmo bloco, ele tentará gravar a média com a chave `sensor123_10h00` de novo, simplesmente sobrescrevendo o mesmo local no banco NoSQL.

## 6. Exemplo Simples de Código
```sql
-- Em bancos chave-valor como DynamoDB ou Cassandra, usar atributos imutáveis 
-- como Chave Primária é suficiente para garantir idempotência.

CREATE TABLE sessoes (
    session_id BIGINT, -- ID gerado de forma determinística via HASH
    user_id BIGINT,
    ingestion_time TIMESTAMP,
    PRIMARY KEY(session_id, user_id)
);
```

## 7. Padrões Relacionados ou Nomes Similares
Usa lógicas do padrão *Windowed Deduplicator* para estabelecer chaves previsíveis e é uma alternativa simples ao *Transactional Writer* no universo streaming.
