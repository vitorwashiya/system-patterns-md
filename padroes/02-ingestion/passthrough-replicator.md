# Padrão: Replicador de Passagem (Passthrough Replicator)

## 1. Resumo (O que é?)
O Replicador de Passagem é um padrão responsável por copiar dados exatamente como eles são (as-is) de um ambiente ou sistema para outro, sem realizar qualquer tipo de modificação ou transformação lógica.

É como "tirar um xerox" de um conjunto de dados de produção para que ele fique acessível em outros ambientes ou serviços, preservando totalmente seu estado original, tipo e estrutura.

## 2. O Problema
* Você precisa sincronizar um conjunto de dados entre o ambiente de produção e o ambiente de testes/desenvolvimento para facilitar a detecção de bugs.
* A fonte original de dados não é idempotente (ex: uma API que pode retornar resultados diferentes a cada chamada), então você não pode apenas rodar a extração duas vezes; você precisa replicar os dados exatos de produção.

## 3. A Solução
Ao adotar o Replicador de Passagem, você implementa um fluxo focado puramente em leitura e escrita. Isso pode ser feito através de comandos de sincronização na infraestrutura (como replicação de buckets em provedores de nuvem) ou usando rotinas simples de cópia na camada de processamento. O foco é manter o formato idêntico ao de origem para evitar problemas silenciosos de tipagem.

## 4. Consequências e Trade-offs
* **Vantagens:** Preserva a integridade total do dado (garantia de espelho); simples e barato de implementar com utilitários de sistema e scripts.
* **Desvantagens/Atenção:** 
  * **Segurança de Dados Pessoais (PII):** Como ele copia tudo, existe um alto risco de vazar dados sensíveis de produção para ambientes de menor segurança. Se existirem dados pessoais, não use este padrão.
  * **Isolamento de Ambiente:** É recomendado usar uma abordagem de envio (push) a partir da produção em vez de coleta (pull), garantindo que problemas em ambientes inferiores não sobrecarreguem ou afetem a produção.

## 5. Exemplo de Aplicação Prática
Você possui arquivos de log crus (JSON) armazenados em um bucket na nuvem de produção e deseja treinar um modelo de aprendizado de máquina na nuvem de testes. Você configura uma rotina de sincronização noturna via script de infraestrutura que simplesmente copia e cola os arquivos JSON de um lado para o outro.

## 6. Exemplo Simples de Código
```bash
# Sincronização direta via linha de comando (AWS CLI) sem transformar o dado
aws s3 sync s3://bucket-producao/dados/ s3://bucket-testes/dados/ --delete
```

## 7. Padrões Relacionados ou Nomes Similares
Sinônimos incluem *Data Copy*, *Data Replication* e *Mirroring*. Pode atuar como a fundação para o padrão *Transformation Replicator* caso dados sensíveis necessitem ser mascarados.
