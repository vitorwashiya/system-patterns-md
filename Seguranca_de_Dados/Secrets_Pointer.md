# Secrets Pointer

**Categoria:** Segurança de Dados

## 🎯 Objetivo
Salvaguardar implacavelmente credenciais lógicas isolando conexões puras e hardcodes nefastos expostos através do emprego rigoroso nativo da referência orquestradora blindada das clouds, evitando a infiltração de palavras-passe na configuração.

## O Problema
Tens repositórios GIT que albergam lógicas transacionais em batch da cloud e APIs para analítica nas pipelines purificadas estritas em produção. Sem pensares muito codificaste ativamente lá no script as passwords. Houve leaks nos acessos no GIT base e faturaram na plataforma originária alheia externa da base consumos enormes puros nos logs não previstos nas tuas requisições. Para atenuar proíbes desde logo chaves estáticas (Hardcodes) nas scripts no repo nativo da equipa.

## A Solução
Perante o óbvio e purificando orquestrações isolas o fluxo ativando a frente nativa e infalível "Secrets Pointer". Usas o modelo puro a apontar em código (`Pointers estritos`) no Spark ou no orquestrador invocando de imediato e ativamente Serviços centrais geridos pelas frameworks das nuvens das infraestruturas base (`GCP Secret Manager` ou `AWS Secrets Manager`). Colocas na gestão do cloud seguro e isolado nas chaves primárias o conteúdo base e os consumidores ativamente nas conexões nativas do seu pipeline interrogam remotamente as URIs do apontador na Cloud estrita no momento originário purificado `secretsmanager_client.get_secret_value()` para carregar localmente e no imediato em runtime (memória nativa das pipelines operacionais das ligações). Os segredos base nunca são revelados no repositório estrito da equipa originária das tabelas em scripts.

## Prós e Contras
- **Prós:** 
  - Centraliza estrondosamente e perfeitamente num repositório imune o acesso seguro na auditoria facilitando trocas unificadas purificadas nativas ativas no ambiente único sem afetar e reiniciar múltiplos repositórios GIT nativos paralelos orquestrados nas APIs locais originais no código em execução estrita de múltiplos componentes das tabelas em nuvens.
- **Contras:** 
  - Requer perante execuções de Streaming pipelines nativos cuidados complexos exímios em invalidar chaves (Cache Invalidation e Rotate keys) que forçam restarts parciais de aplicações long-running ou mecanismos morosos de restauro dinâmicos assíncronos originários das conexões e rotinas nas reconexões.
  - Oferece amiúde nos logs de execução purificados nativamente orquestrados o falso sentido protetor estrito nas chaves, visto que um `print()` do programador mal intencionado na framework exporá inadvertidamente o valor carregado nas consolas ou visualizações purificadas das instâncias orquestradoras (Vazamentos purificados do contexto).