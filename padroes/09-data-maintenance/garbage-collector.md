# Padrão: Coletor de Lixo (Garbage Collector)

## 1. Resumo (O que é?)
O padrão de Coletor de Lixo (*Garbage Collector*) automatiza as regras temporais e políticas corporativas de governança de retenção de dados (Data Retention Policies), apagando massivamente dados úteis (não metadados) que atingiram o limite de vida legal ou analítico e que não devem mais existir em sistemas quentes.

## 2. O Problema
* O seu Data Lake acumula transações há 15 anos. Mas o banco só permite analisar e reportar na ferramenta dados dos últimos 5 anos.
* Manter 10 anos de dados mortos na S3 na classe "Standard" custa US$ 40 mil dólares extras por mês.

## 3. A Solução
Você implementa a Coleta de Lixo alavancando os recursos nativos dos provedores de objetos. Você não cria um Job PySpark para fazer `DELETE WHERE ano < 2018`, pois isso envolveria custos de computação e leitura que você quer evitar. Em vez disso, você configura um Lifecycle Policy (*Política de Ciclo de Vida*) direto na infraestrutura do Bucket S3 (ou Azure Blob). O servidor, em plano de fundo sem uso do seu hardware, varrerá as raízes horizontais de partições (`/ano=2017/`) e excluirá os arquivos permanentemente quando baterem a idade estipulada, ou os moverá silenciosamente para a "Geladeira" (como o Amazon Glacier) cobrando centavos.

## 4. Consequências e Trade-offs
* **Vantagens:** Salva a fatura e o orçamento financeiro de longo prazo; atua como segurança anti-auditoria, pois o dado some automaticamente, impedindo usos não conformes com o tempo.
* **Desvantagens/Atenção:** 
  * **Exclusões Catastróficas Intratáveis:** Regras configuradas de forma errada na nuvem varrerão sua empresa inteira da face da Terra em 1 dia, sem que orquestradores ou bancos tenham a chance de dar erro, pois o Coletor de Lixo é uma barreira de infraestrutura silenciosa.
  * **Sincronia com Metadados:** Se a S3 apagar os arquivos físicos e o catálogo do Big Data não for notificado, quando o Spark tentar listar os meses antigos, ele sofrerá "FileNotFoundException", falhando bizarramente os processos corporativos até que a tabela *External Table* seja reparada.

## 5. Exemplo de Aplicação Prática
Para os logs crus da aplicação que chegam por *Object Storage* JSON, a equipe do AWS aplica uma política via infraestrutura. Passados 30 dias de criação, o arquivo vai pro Glacier. Passados 365 dias, o Glacier vaporiza a informação. Os engenheiros da equipe sequer precisam lembrar que esse processo existe.

## 6. Exemplo Simples de Código
```hcl
# Regra de Terraform injetando Garbage Collection direto na Nuvem (AWS S3 Lifecycle)
resource "aws_s3_bucket_lifecycle_configuration" "regra_limpeza_logs" {
  bucket = aws_s3_bucket.logs.id

  rule {
    id     = "excluir_logs_apos_1_ano"
    status = "Enabled"

    filter {
      prefix = "logs-aplicacao/"
    }
    
    # Destrói irremediavelmente (Coleta de Lixo Final) após 365 dias
    expiration {
      days = 365
    }
  }
}
```

## 7. Padrões Relacionados ou Nomes Similares
Muitas vezes integrado nativamente ao conceito primário do armazenamento *Object Storage*. Muito diferente da limpeza de logs e metadados regida pelo *Metadata Purger*.
