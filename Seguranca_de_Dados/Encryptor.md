# Encryptor

**Categoria:** Segurança de Dados

## 🎯 Objetivo
Proteger incondicionalmente a base repousada nos sistemas (data at rest) e a base que transita na rede purificada subjacente (data in transit) de roubos informativos em ataques físicos por desvio de acessos de provedor tornando os pacotes incompreensíveis nos discos originários ou na extração não legitimada ativa do log.

## O Problema
Com os controlos de acesso a funcionar estritamente a direção da tua infraestrutura está paranoica. Temem ativamente interceções em trânsito entre nós nos brokers das máquinas da rede na nuvem partilhada (Cloud) originária ou, de forma insidiosa estrita, receiam um desvio não autorizado a extrair fisicamente ou aceder estritamente num bypass bruto e abstrato ao disco que atenda perfeitamente o dump nativo no S3 sem decifrações explícitas de identidade no DW e sem permissão e contexto nos limites analíticos.

## A Solução
Acionas ativamente perante estas duas vertentes e blindas organicamente sob os trâmites fundamentais no padrão Encryptor. Em repouso estrito invocas serviços gerenciados na nuvem (KMS Key Management, Azure Vaults), declarando explicitamente a cifragem nas escritas. O serviço cifra nativamente no Server-Side no momento de injeção ou delegas nas Client-Side do teu pipeline a ofuscação pura via chaves partilhadas geridas isoladamente orquestrando acesso restrito ao próprio recurso criptográfico (que bloqueia o disco decifrado a chaves exclusivas de APIs geridas nativas autorizadas). Em paralelo e organicamente na fase transitiva (data-in-motion), garantes nos SDKs purificados ligações nas bases atualizando semáforos dos protocolos purificados TLS nas invocações forçadas na base de APIs externas no percurso de brokers do Kafka ativamente nas portas isoladas de SSL originário puro (ex. TLS 1.2 minimum version na rede cloud estrita na cloud nativa originária purificada restrita).

## Prós e Contras
- **Prós:** 
  - Elimina de imediato sem esforço visível do lado da API analítica atrito de roubo da infraestrutura física, suportando normativos e garantias estritas invioláveis na Cloud perante entidades audíveis de base.
  - Abstrai incrivelmente os trabalhos dos engenheiros, baseando totalmente o processo de decifragem do lado Server gerido via KMS nos SDK das Clouds que mascaram processos base originários em background com o serviço central.
- **Contras:** 
  - Impõe overhead considerável nos ciclos de processador (CPU) em leituras e injeções estritas do lado das pontas analíticas e encarece ligeiramente os fluxos transacionados nos armazenamentos em repouso da base com a latência acrescida do protocolo SSL nativo puro e forte na infraestrutura na Cloud transacional originária (Decription/encryption phases nos IOs).
  - Se perderes o acessos às KMS ou apagares as chaves vitais sem garantias de recuo de Soft-Delete (KMS Grace periods puros) perdes irrevogavelmente toda a tua infraestrutura em disco analítico sem volta atrás nos ficheiros nativamente selados nos Data Lakes (Risco brutal puro dos Lockouts totais dos ativos lidos).