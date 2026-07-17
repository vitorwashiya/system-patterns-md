# Padrão: Pseudo-Anonimizador (Pseudo-Anonymizer)

## 1. Resumo (O que é?)
Enquanto o Anonimizador destrói a identidade, o Pseudo-Anonimizador atua mascarando, criptografando ou "tokenizando" a informação pessoal em um valor substituto rastreável. Seu foco principal é permitir o processamento seguro do registro na vasta maioria dos ambientes, mas guardando uma chave secreta e altamente restrita em outro lugar para *re-identificar* o usuário (reverter o disfarce) quando regras estritas e pontuais do negócio exigirem.

## 2. O Problema
* O e-commerce terceiriza a prevenção a fraude. Eles mandam bilhões de e-mails diários de clientes para um sistema na nuvem cruzar riscos. Enviar e-mails verdadeiros fere as leis locais. 
* Destruir os e-mails (Anonimização) com `NULL` resolveria a privacidade, mas estragaria a inteligência (o algoritmo de fraude não conseguiria verificar se aquele e-mail sempre cai em fraudes). Como compartilhar segurança sem expor?

## 3. A Solução
Aplicar o Pseudo-Anonimizador usando **Tokenização** ou **Criptografia Simétrica Determinística**. Em ambas, quando a pipeline de ingestão lê o dado real `joao@email.com`, ela invoca um serviço criptográfico centralizado fechado (Vault/Cofre). O Vault substitui o e-mail por um Token ininteligível (ex: `A93Bx`). Todas as dezenas de tabelas de Big Data, Marketing, e Análises são populadas com `A93Bx`. O sistema externo faz cálculos cruzando os tokens perfeitamente (pois o mesmo e-mail sempre vira o mesmo token determinístico). Quando uma fraude é detectada e um e-mail policial precisa ser despachado, um processo com permissão Máxima (Vault Admin) submete o `A93Bx` na API do cofre e ela "devolve" `joao@email.com` apenas para aquela requisição (re-identificação).

## 4. Consequências e Trade-offs
* **Vantagens:** O equilíbrio perfeito entre a utilidade brutal (agrupamentos, analítica) e o respeito aos requisitos de governança, mantendo as vias corporativas limpas.
* **Desvantagens/Atenção:** 
  * **Ponto Único de Falha (SPOF) e Desempenho:** Todo e qualquer novo registro de ingestão em streaming ou batch deve ser disparado em alta velocidade pela rede para bater num cofre (Ex: AWS KMS / Hashicorp Vault) gerar a conversão e voltar, afogando os cofres corporativos e inflando a latência.
  * **Obrigações Jurídicas Contínuas:** Diferente do dado anônimo absoluto, dados pseudo-anonimizados (que podem ser revertidos por funcionários autorizados) **são juridicamente dados pessoais** na visão de reguladores. Logo, os discos onde os tokens repousam devem obedecer exclusões e LGPDs estritamente.

## 5. Exemplo de Aplicação Prática
Para não ter infraestrutura própria lidando com cartão de crédito (PCI-DSS), a empresa do aplicativo usa o *Tokenizador* do serviço Stripe. Quando o usuário digita o cartão na tela final, o Gateway captura os números e devolve para o banco de dados interno da empresa apenas `tok_1J2b...`. A empresa usa o token em seus Big Datas, mas nunca tem a capacidade de imprimir os números originais no plástico.

## 6. Exemplo Simples de Código
```python
# Pseudo-anonimização simulada (Criptografia Determinística por Chave Central)
from cryptography.fernet import Fernet
# Chave segura mantida apenas pelo Vault do Admin
chave_sistema = b'qRw-_h...='
cipher_suite = Fernet(chave_sistema)

# Todos as tabelas corporativas recebem e processam "dado_pseudo" cegamente
dado_pseudo = cipher_suite.encrypt(b"joao@mail.com")

# Quando a lei manda, o administrador com a 'chave_sistema' reverte
dado_recuperado = cipher_suite.decrypt(dado_pseudo) 
```

## 7. Padrões Relacionados ou Nomes Similares
Base dos sistemas de *Tokenization* modernos para meios de pagamento (PCI-Compliance). Pode ser operado localmente em banco utilizando algoritmos de *Hashing* com sal (Salt) gerenciado (Dynamic Salting).
