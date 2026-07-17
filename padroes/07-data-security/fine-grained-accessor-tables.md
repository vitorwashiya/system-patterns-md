# Padrão: Acessador de Tabela de Granularidade Fina (Fine-Grained Accessor for Tables)

## 1. Resumo (O que é?)
O Acessador de Tabela de Granularidade Fina (*Fine-Grained Accessor*) é um padrão de Controle de Acesso Analítico onde restrições e filtros de segurança rigorosos são aplicados não à tabela como um todo, mas sim a colunas e linhas específicas (Column-level security e Row-level security), ocultando-as em tempo de leitura de acordo com quem está consultando.

## 2. O Problema
* Em uma arquitetura moderna (Data Lakehouse), todos os analistas corporativos acessam o sistema via a mesma tabela `vendas_brutas`. 
* A equipe de contabilidade precisa ver todas as linhas para consolidar as receitas. Porém, vendedores só devem poder visualizar as colunas e as linhas de vendas que **eles próprios** realizaram, sendo cegos aos clientes das outras filiais e a certos campos como *CPF* do comprador. 

## 3. A Solução
Ativar no Banco de Dados a "Segurança a nível de Coluna" (negando acesso a propriedades inteiras) e "Segurança a nível de Linha" (negando as entidades da matriz). Na segurança de coluna, os bancos permitem o comando `GRANT SELECT(col_A, col_B) TO perfil`, ou permitem uso de mascaramento (Masking Functions). Na segurança de linha, utiliza-se diretivas de metadados como as `Row Access Policies` para atar silenciosamente a sessão ativa de um usuário `X` num comando implícito (`WHERE vendedor_id = 'X'`), cortando fora na raiz registros de outros perfis no instante da consulta (SELECT) transparente.

## 4. Consequências e Trade-offs
* **Vantagens:** Otimização brutal da gestão de segurança. Elimina a necessidade de criar (e gastar armazenamento para manter) 30 tabelas isoladas para os 30 vendedores diferentes ou Views infinitas. 
* **Desvantagens/Atenção:** 
  * **Tipagem dos Dados:** Funções dinâmicas de mascaramento de coluna podem não se dar bem nativamente em estruturas altamente aninhadas (como Arrays ou JSON complexos na coluna) ou corromper a inferência de tipo na resposta para o cliente.
  * **Custos na Consulta (Query Overhead):** Toda a mágica acontece executando dezenas de IF-ELSES sistêmicos ou subconsultas gigantes silenciosas e metadados lógicos para avaliar "o usuário é membro do grupo X?" atrás do SELECT do cliente, o que pode cobrar o seu preço no faturamento do tempo computacional global (latência oculta do motor).

## 5. Exemplo de Aplicação Prática
Um médico loga no hospital e procura na base de pacientes o termo "Gripe". O banco de dados analisa os metadados lógicos de login dele, descobre que é Pediatra, e silenciosamente corta a base, retornando pacientes curados apenas da área pediátrica. Ao mesmo tempo, ele não traz a coluna de "plano de saúde", pois o hospital configurou um mascaramento na coluna porque o médico só deveria saber do quadro clínico, não financeiro.

## 6. Exemplo Simples de Código
```sql
-- Segurança no PostgreSQL: Acesso de Linha
-- Liga a política implícita invisível: Quem pesquisar só verá linhas onde a coluna 'login' for igual ao usuário ativamente logado rodando a query
ALTER TABLE relatorio_pediatria ENABLE ROW LEVEL SECURITY;
CREATE POLICY dr_apenas_acessa_dele ON relatorio_pediatria USING (login = current_user);

-- Segurança no PostgreSQL: Acesso de Coluna Limitado
GRANT SELECT(paciente_id, batimento_cardiaco, sintomas) ON relatorio_pediatria TO dr_joao;
```

## 7. Padrões Relacionados ou Nomes Similares
Comumente engloba conceitos nativos das marcas como as *Row Level Security (RLS)* ou *Column Level Security (CLS)* e *Policy Tags* (BigQuery). Relaciona-se com soluções manuais geradas pelo *Dataset Materializer*.
