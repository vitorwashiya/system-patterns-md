# Padrão: Replicador com Transformação (Transformation Replicator)

## 1. Resumo (O que é?)
O Replicador com Transformação copia dados de um ambiente para outro, mas adiciona uma camada intermediária de transformação (como remoção ou mascaramento de campos). Diferente do Replicador de Passagem, ele altera intencionalmente o conjunto de dados durante o trânsito.

Esse padrão é muito utilizado quando você precisa de dados de produção para testes em ambientes de homologação (staging), mas a regulação de privacidade o impede de copiar as informações sensíveis (PII) originais.

## 2. O Problema
* Você quer usar dados reais para testar seus pipelines antes do lançamento em produção, pois geradores de dados sintéticos não simulam os problemas reais de qualidade de dados.
* Os dados de produção contêm informações de identificação pessoal (PII) que não podem ser levadas para ambientes de desenvolvimento ou staging por razões de conformidade.

## 3. A Solução
Implementar o padrão Replicador com Transformação significa criar um fluxo com três etapas: ler os dados, aplicar uma transformação e escrevê-los no destino. A transformação substitui ou remove atributos indesejados e pode ser feita com mapeamentos customizados (usando bibliotecas como Apache Spark) ou com instruções SQL (`SELECT * EXCEPT(...)`).

## 4. Consequências e Trade-offs
* **Vantagens:** Permite realizar testes precisos usando a estrutura e volume dos dados de produção, sem comprometer a segurança e a privacidade.
* **Desvantagens/Atenção:** 
  * **Risco de Transformação e Dessincronização:** É necessário muito cuidado ao transformar arquivos de texto bruto (como JSON ou CSV), pois falhas na definição de tipos (ex: timestamps) podem quebrar o schema silenciosamente.
  * **Evolução de Dados:** O catálogo de quais campos são PII pode evoluir. A lógica precisa ser constantemente revisada ou integrada a um sistema de Governança de Dados para evitar o vazamento acidental de novos campos privados.

## 5. Exemplo de Aplicação Prática
Para homologar um novo cálculo de frete, a equipe precisa do histórico de compras. Um job diário lê os dados de produção, apaga a coluna de `nome_cliente` e `endereco_exato`, mantendo apenas o `cep` e `valor`, e salva isso em um ambiente de testes que a equipe de QA pode acessar com segurança.

## 6. Exemplo Simples de Código
```sql
-- Exemplo de como usar SQL para copiar uma tabela removendo colunas sensíveis
SELECT * EXCEPT (ip, latitude, longitude) 
FROM producao.tabela_usuarios
```

## 7. Padrões Relacionados ou Nomes Similares
Uma evolução mais segura do *Passthrough Replicator*. Pode usar o padrão *Anonymizer* (Anonimizador) como função dentro do fluxo para mascarar os dados que não podem ser apenas deletados.
