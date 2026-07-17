# Padrão: Envelope (Wrapper)

## 1. Resumo (O que é?)
O padrão de Envelope (Wrapper) é um padrão de design de representação e decoração de dados. Ele adiciona comportamentos, metadados ou informações processadas a um registro, colocando tudo isso "ao redor" (envelopando) dos dados brutos originais, que permanecem intocados dentro da nova estrutura.

## 2. O Problema
* Seu pipeline consolida eventos vindo de múltiplas fontes, todas com esquemas (schemas) ligeiramente diferentes.
* Você quer extrair campos importantes ou gerar valores enriquecidos, mas não quer destruir ou alterar a estrutura original da mensagem, pois usuários a jusante (como auditores) precisam debugar como a mensagem chegou originalmente.

## 3. A Solução
Você define uma abstração no nível do registro. Essa abstração envolve os valores originais em um envelope de alto nível. Além do payload original não modificado (geralmente jogado dentro de um objeto aninhado como `raw_data`), o envelope inclui os atributos enriquecidos, computados e o contexto técnico (como tempo de processamento ou versão do pipeline). Você pode implementar com estruturas aninhadas (`STRUCT` em bancos modernos) ou achatando (denormalizando) as tabelas lado a lado.

## 4. Consequências e Trade-offs
* **Vantagens:** Preservação total do histórico original, clareza e divisão entre o que é "dado puro" e o que é "enriquecimento do pipeline". Facilita auditorias e debug.
* **Desvantagens/Atenção:** 
  * **Divisão de Domínio:** Alguns atributos lógicos podem acabar duplicados ou divididos entre a camada original e a camada envelopada, o que exige boa documentação para não confundir os consumidores.
  * **Tamanho do Payload:** Valores envelopados se tornam parte intrínseca de todos os registros processados, inflando o tamanho geral do dado armazenado e do tráfego de rede. (Para amenizar, opte por formatos colunares como Parquet).

## 5. Exemplo de Aplicação Prática
Um evento chega no sistema com dezenas de colunas esotéricas de tracking. O pipeline de dados envelopa essa mensagem criando um schema estruturado onde a raiz tem colunas úteis como `user_id_computado` e `data_padronizada`. Todo o evento inicial, sem nenhuma modificação, fica preservado dentro de um campo de texto (ou JSON aninhado) chamado `payload_original`.

## 6. Exemplo Simples de Código
```sql
-- Criando um Wrapper através de SQL, isolando a versão original no campo 'raw'
SELECT 
    context.user.connected_since IS NOT NULL as is_connected,
    STRUCT(visit_id, event_time, user_id, page, context) AS raw
FROM input_visits
```

## 7. Padrões Relacionados ou Nomes Similares
Parecido visualmente com o *Metadata Decorator* (Decorador de Metadados), mas o Envelope atua misturando regras de negócio públicas, e não apenas informações puramente técnicas e invisíveis.
