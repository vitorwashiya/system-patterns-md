# Padrão: Criptografador (Encryptor)

## 1. Resumo (O que é?)
O padrão Criptografador atua na barreira absoluta contra sequestro de dados (Data Intrusion) forçando o dado a se tornar uma confusão ilegível caso as regras do orquestrador, do banco de dados e as travas de infraestrutura sejam quebradas. Trabalha em duas vertentes: Protegendo o dado que está adormecido no servidor (At-Rest) e protegendo o dado que viaja no cabo de rede (In-Transit).

## 2. O Problema
* Os dados pessoais de milhões de clientes (PII) no data warehouse já estão sob rigorosas travas Cloud (`Fine-Grained Accessor`), mas e se o administrador corporativo baixar acidentalmente os discos do disco rígido da nuvem (Storage)?
* E se *hackers* capturarem e decodificarem pacotes perdidos na fibra óptica transitando do Kafka até o Spark, burlando todas as senhas virtuais?

## 3. A Solução
Empregando o **Criptografador**. 
1. **Em Repouso (At-Rest):** A nuvem usa chaves centrais gerenciadas externamente como as do cofre KMS (AWS/GCP KMS, Azure Vault). Sempre que um produtor envia dados (Client-Side), ou que o próprio banco os registra fisicamente nas partes metálicas do seu disco rígidio em segundo plano (Server-Side), o KMS é acionado secretamente para misturar/embaralhar as letras do arquivo nativo, que nunca existirá livre sem a chave simétrica.
2. **Em Trânsito (In-Transit):** Você delega atualizações de TLS (Transport Layer Security) modernos configurando seu servidor para ignorar tráfegos passados sob HTTP exposto ou sem certificados de chave pública.

## 4. Consequências e Trade-offs
* **Vantagens:** Atende diretamente à obediência severa a frameworks jurídicos e auditorias como SOX ou HIPAA. Torna qualquer dado vazado em lixo matemático sem valor econômico aos infratores.
* **Desvantagens/Atenção:** 
  * **Overhead Computacional Lento:** Desembaralhar a cada SELECT e embaralhar a cada UPDATE deprime os ciclos brutais de processamento na nuvem consumindo muito hardware.
  * **Risco Massivo de Perda Fatal:** O sequestro vira contra o dono do cofre. Diferente do dado desprotegido, se a sua corporação por falha técnica "Perder" o cofre do serviço KMS com a chave principal de leitura (se alguém acidentalmente apagar lá dentro), centenas de Data Lakes gigantes instantaneamente tornam-se cacos indecifráveis até o fim dos tempos. Nuvens evitam isso adotando o conceito de Janela de *Soft Deletes* que não deletam senhas no cofre sem que se passem de 7 a 30 dias de aviso.

## 5. Exemplo de Aplicação Prática
O RH possui um Bucket da S3 chamado `/salarios/`. O arquiteto usa Terraform para forçar o serviço Amazon S3 a aplicar encriptação AWS KMS com a regra `sse_algorithm = aws:kms`. Agora, se porventura ocorrer falha bizarra da Amazon S3 e a pasta do RH cair aberta ao público da web global (Breech), a internet não lerá a planilha pois não possuem fisicamente acesso ao módulo KMS, exigido para dar sentido ao código misturado.

## 6. Exemplo Simples de Código
```hcl
# Exemplo Infra-As-Code (Terraform) obrigando KMS no Bucket AWS S3
resource "aws_s3_bucket_server_side_encryption_configuration" "visitas" {
  bucket = aws_s3_bucket.visitas.id
  rule {
    apply_server_side_encryption_by_default {
      kms_master_key_id = module.kms.key_arn
      sse_algorithm     = "aws:kms"
    }
  }
}
```

## 7. Padrões Relacionados ou Nomes Similares
Uma fundação primária de Defesa Em Profundidade, juntamente com o *Fine-Grained Accessor for Resources*. Conhecida nos meios populares como criptografia *Server-Side Encryption (SSE)* ou de Lado do Cliente.
