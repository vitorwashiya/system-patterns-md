# Padrão: Apontador de Segredos (Secrets Pointer)

## 1. Resumo (O que é?)
O padrão Apontador de Segredos foca na remoção absoluta de variáveis sensíveis e credenciais literais gravadas (Hard-coded) no código de implantação da aplicação e dos fluxos. Em vez disso, o código usa uma mera referência (um Apontador / Pointer) sem valor para o hacker. Essa referência dita onde o cofre injetará a credencial secreta da conta final no instante em que o sistema subir para a memória (Run-time).

## 2. O Problema
* Seu pipeline está rodando, e nele tem o seu `banco_password = "senha123"`. Como você faz controle de versão no Git, a senha é acidentalmente versionada (commit) e exposta para 50 pessoas da empresa no Github. 
* Um bot raspador de senhas vasculha os arquivos e clona seu banco usando suas credenciais textuais expostas no GitHub corporativo.

## 3. A Solução
Utilizar a camada de um "Cofre de Segredos" (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault).
1. Você deleta `senha123` do código de vez e loga no painel virtual do cofre, cadastrando-a como uma chave "prod_banco_pwd". 
2. Você injeta o "Apontador" ou variáveis de ambiente nos arquivos de projeto (ex: `password = os.environ("PROD_BANCO_PWD")`).
3. Quando a máquina ou container vai ao ar, um processo seguro com papéis de IAM estritos (como o *Fine-Grained Accessor for Resources*) visita o cofre na calada, coleta a senha `senha123` e a coloca transitoriamente na RAM apenas daquela máquina rodando o script. 

## 4. Consequências e Trade-offs
* **Vantagens:** Erradica o vazamento visual e auditável de senhas corporativas. Permite que as equipes mudem (Rotacionem) a senha original no cofre de 3 em 3 meses sem encostar em uma linha do script do desenvolvedor ou precisarem de "re-deploy".
* **Desvantagens/Atenção:** 
  * **Custo Adicional Oculto:** Provedores de nuvem muitas vezes cobram por cada solicitação ao Secrets Manager. Jobs escalados em processamento distribuído (Ex: Spark rodando em 500 nós paralelos) poderiam causar 500 mil requisições num loop e gerar milhares de dólares apenas lendo a senha.
  * **Dependência (Vendor Lock-in):** Você fica amarrado e restrito ao fluxo exato do modelo de apontador do seu provedor, que difere radicalmente do Google para AWS ou Azure.

## 5. Exemplo de Aplicação Prática
Para implantar o fluxo do Airflow sem vazar os acessos da rede Snowflake da empresa, você salva tudo no "Azure Key Vault". No Airflow, o *Dev* apenas define o apontador: cria a connection `snowflake_prod` mas passa a URI URI literal do Vault. O Airflow resolve sozinho.

## 6. Exemplo Simples de Código
```python
# Pipeline Python buscando segredos na hora de rodar via Pointer Boto3 (AWS)
import boto3
import json

cliente_cofre = boto3.client('secretsmanager')
# Apontador. O hacker no Github que ver esse código lerá isso e nada fará
ponteiro = "producao/banco_dados/main"

resposta = cliente_cofre.get_secret_value(SecretId=ponteiro)
senha_real_invisivel = json.loads(resposta['SecretString'])['password']

db.connect(pwd=senha_real_invisivel)
```

## 7. Padrões Relacionados ou Nomes Similares
Implementações frequentes de *Infrastructure as Code (IaC)*. Pode vir pareado das técnicas base de *Secretless Connector*.
