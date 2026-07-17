# Padrão: Acessador de Recurso de Granularidade Fina (Fine-Grained Accessor for Resources)

## 1. Resumo (O que é?)
O Acessador de Recurso de Granularidade Fina delega o controle de acesso de segurança pesada para a nuvem em que o dado reside, atribuindo a uma identidade estrita a habilidade exata e limitada para o funcionamento lógico do sistema (Princípio do Menor Privilégio / *Least Privilege*). É a trava definitiva da arquitetura Cloud.

## 2. O Problema
* Auditores relataram que as "Tarefas Automatizadas do Servidor A" usam uma chave de acesso genérica da nuvem chamada `AcessoGeralDados` que as permite ler, deletar e sobrescrever todo e qualquer repositório (Bucket S3) ou Fila de Mensagens da empresa.
* Se um script malicioso invadir essa Tarefa Automatizada, ele poderia sequestrar todos os bancos sem impedimento da arquitetura.

## 3. A Solução
Em vez de depender das credenciais frouxas, você alavanca a funcionalidade do provedor Cloud, o IAM (Identity and Access Management / Gerenciamento de Acessos). Nele você deve escrever um contrato (Policy JSON) incrivelmente rígido: "O script de análise de telemetria ganha uma identificação própria. Essa identidade SÓ pode executar `Leitura` (S3:GetObject) E SOMENTE dentro da pasta `S3://arquivos/telemetria/2024/`. O `DELETE` está PROIBIDO". As nuvens resolvem o padrão de duas formas atreláveis:
1. **Baseado no Recurso:** Ancorado dentro da configuração nativa do repositório/bucket de dados (impedindo externamente as conexões).
2. **Baseado na Identidade:** Ancorado no Cargo/Identity (*IAM Role*) das ferramentas virtuais ou de usuários.

## 4. Consequências e Trade-offs
* **Vantagens:** O grau mais poderoso de defesa cega contra vulnerabilidade. Garante a proteção imutável e inalienável da base.
* **Desvantagens/Atenção:** 
  * **Cota de Limites:** Sistemas como o IAM da nuvem podem ter limites lógicos restritos de políticas customizadas criadas para sua corporação.
  * **Pesadelo Operacional:** Esculpir políticas em código JSON com caminhos literais cria uma infinidade massiva de arquivos IAM impossíveis de gerenciar ou entender. Você pode atenuar a granularidade utilizando atalhos e curingas como caracteres de coringas lógicos (`*`) no prefixo de permissão `s3://logs/visitas/*`, mas afrouxa perigosamente a lei de Segurança dos Livros.

## 5. Exemplo de Aplicação Prática
Um job diário processador roda num Apache EMR da Amazon e puxa informações puras da frota. Você cria um documento de acesso digital (Role), conecta ao EMR, e nesse papel amarra regras que ele só tem permissão de leitura nos tópicos do `Kinesis Data Streams`. O código interno acidentalmente vaza um bug usando `PutRecord` tentando gravar de volta na rede; a Amazon barra automaticamente na porta através das diretivas cegas.

## 6. Exemplo Simples de Código
```json
// JSON na linguagem Cloud de Controle de Acesso (Ex: Amazon IAM S3 Policy)
// "Para as chaves com este usuário, permita ler apenas os dados do bucket-visitas"
{
  "Statement": [{
    "Effect": "Allow", 
    "Action": ["s3:Get*", "s3:List*"],
    "Resource": ["arn:aws:s3:::bucket-visitas-protegido/*"]
  }]
}
```

## 7. Padrões Relacionados ou Nomes Similares
Mais amplamente reconhecido como aderência rigorosa de *IAM Policies* focadas no *Least-Privilege*. Alternativa fundamental baseada em repositórios em contraponto às camadas lógicas em banco (tabelas) do *Fine-Grained Accessor for Tables*.
