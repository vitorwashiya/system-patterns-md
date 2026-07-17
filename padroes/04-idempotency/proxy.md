# Padrão: Proxy (Proxy)

## 1. Resumo (O que é?)
O padrão Proxy é uma abordagem que resolve o problema de mutabilidade adicionando um nível extra de indireção ("um intermediário") entre o conjunto de dados físico e os usuários finais. Dessa forma, você atende a requisitos legais de dados imutáveis enquanto apresenta sempre a versão mais recente para o consumidor de forma transparente.

## 2. O Problema
* Seu pipeline sempre sobrescreveu o conjunto de dados antigo com um novo porque você só precisava da versão mais recente.
* Mas agora o departamento Jurídico exigiu que você guarde o histórico exato de todas as versões antigas geradas para auditoria (arquivos imutáveis).
* Você não pode quebrar os painéis dos consumidores que esperam ver apenas os dados novos consolidados.

## 3. A Solução
Para garantir a imutabilidade exigida pela auditoria legal, o pipeline não deve mais sobrescrever tabelas, e sim gerar os dados em locais novos e protegidos (como tabelas com carimbo de tempo ou arquivos *WORM - Write Once Read Many* bloqueados via nuvem). Para não sobrecarregar os consumidores, você cria uma visualização (View) ou um manifesto (como um "Proxy"). Esse Proxy atua como a única porta de entrada exposta ao usuário e sempre apontará apenas para a tabela ou arquivo gerado mais recentemente.

## 4. Consequências e Trade-offs
* **Vantagens:** Resolve o atrito entre arquivamento legal/compliance e facilidade analítica. Impede modificações e fraudes em dados históricos.
* **Desvantagens/Atenção:** 
  * **Evolução de Schema:** Se a sua nova versão ganhar um campo obrigatório diferente da antiga, atualizar a `View` (Proxy) pode ser um problema ou exigir que todos os dados históricos sejam adaptados ou que a view possua regras complexas de fallback, fugindo do propósito inicial.
  * **Suporte e Permissões:** Você precisará lidar com travas e bloqueios (locks) em armazenamentos na nuvem para garantir a imutabilidade, o que significa envolver times de infraestrutura e garantir que o próprio script não seja capaz de burlar o bloqueio acidentalmente.

## 5. Exemplo de Aplicação Prática
Para fins de conformidade médica, exames gerados não podem ser deletados. Um script salva os novos processamentos do dia na tabela interna oculta `exames_20241120`, trava as permissões, e em seguida atualiza o código SQL da *View* principal chamada `exames_recentes` para passar a apontar para `exames_20241120`.

## 6. Exemplo Simples de Código
```sql
-- Após o job rodar e criar a tabela 'dispositivos_internos_20240101_120000'
-- Atualizamos a View (Proxy) para apontar para a recém criada.
CREATE OR REPLACE VIEW dedp.dispositivos AS
 SELECT * FROM dispositivos_internos_20240101_120000;
```

## 7. Padrões Relacionados ou Nomes Similares
Padrão muito conectado ao conceito arquitetural de *WORM (Write Once Read Many)*. Pode necessitar de apoio de outros padrões analíticos, como *Fine-Grained Accessor for Resources* para controlar quem pode travar as tabelas no object storage.
