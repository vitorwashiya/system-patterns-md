---
# Passthrough Replicator

## Classificação
- **Tipo:** Estrutural
- **Escopo:** Objeto

## Intenção
Clonar bytes integrais e coleções isoladas através de limites de ambientes sem invocar formatações, avaliações lógicas, ou manipulação de esquemas, assegurando paridade e isonomia fidedigna perfeita em testes.

## Também Conhecido Como
Data Replication.

## Motivação (Contexto)
Fornecer à base de testes em homologação fragmentos de tabelas oficiais imaculadas geradas exaustivamente por APIs instáveis de parceiros sem acionar ou saturar as rotas originárias frágeis que não reagem como idempotentes a invocações constantes.

## Aplicabilidade
- Use o Passthrough Replicator quando:
  - Reprodutibilidades estritamente determinísticas exijam a persistência intocada das cascatas sintáticas passadas.
  - Data Providers não idempotentes proibem extrações re-simuladas confiáveis num limite idêntico temporal.
  - O escopo das entidades prescinde obrigatoriamente ofuscação sensível de dados confidenciais (PII ausente ou aceitável na infraestrutura destino).

## Estrutura e Participantes
- **Storage Source:** Fonte alocadora original guardando logs masterizados de produção irrestrita.
- **Replication Policy:** Diretiva atômica ou rotina subjacente puramente focada no espelhamento físico e cego sem interpretador lógico (S3 Replications / Cloud Policies).
- **Storage Target:** Sincronizador receptor imerso num ambiente analítico lateral.

## Colaborações
Configurações autogeridas ou jobs limitados ao comando nativo "copy" engolem blocos integrais passivos lidos do Storage Source e despacham de maneira linear aos domínios cruzados do Storage Target, isentos de validação formal dos dados encapsulados.

## Consequências
- **Prós:**
  - Isola maravilhosamente a operação de deturpações em transformações equivocadas acidentais (datas em strings formatadas permanecem intocáveis e inalteradas).
  - Maximiza simplicidade conceitual, evitando alocar imensos clusters paralelos de processamento caro.
- **Contras:**
  - Instabilidades cruéis decorrentes das aberturas compulsórias inter-ambientes (Cross-environment risk) se a política originar instabilidades diretas de pânico em redes de sub-nível.
  - Ameaça catastrófica iminente ao migrar acidental e publicamente massas contendo assinaturas privadas de clientes vazadas se adotada descuidadamente.

## Implementação (Exemplo de Código)
```hcl
# Regras nativas operando infraestrutura sem intervenção código lógico de engine (Terraform)
resource "aws_s3_bucket_replication_configuration" "replication_sandbox" {
  role   = aws_iam_role.replication_cross_env.arn
  bucket = aws_s3_bucket.devices_production.id

  rule {
    id = "devices-sync"
    status = "Enabled"
    destination {
      bucket = aws_s3_bucket.devices_staging.arn
      storage_class = "STANDARD"
    }
  }
}
```

## Padrões Relacionados

* **Transformation Replicator:** Herdeiro lógico imediato e compulsório, requisitado em ambientes submetidos às duras regulamentações quando a pura passagem falha legalmente.

---
