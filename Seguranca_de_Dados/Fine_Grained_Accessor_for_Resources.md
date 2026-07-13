# Fine-Grained Accessor for Resources

**Categoria:** Segurança de Dados

## 🎯 Objetivo
Conter incidentes e limitar as ações massivas de destruição e leituras nefastas em provedores Cloud nativos nas camadas infraestruturais estritas obedecendo a imperativos puros corporativos focados nas políticas base de menor privilégio explícito aplicados a instâncias operativas e recursos.

## O Problema
Auditorias do departamento intercetam e assinalam "alerta máximo". Devido aos acessos amplos nos tópicos Cloud base, o script simples da equipa pode de um momento de corrupção infetar ou eliminar na base purificada estrita todas as estruturas de armazenamento "buckets" purificadas de toda a cloud originária. Podes assim obliterar sem dar conta a arquitetura da tua conta corporativa porque o serviço dispõe dos papéis totais ativados indiscriminadamente purificados no cluster por defeito originários.

## A Solução
Assumir sem hesitação o modelo "Least Privilege" e ativar do teu lado do infra-as-code as "Fine-Grained Accessor for Resources". Abordas de forma diametralmente paralela a duas frentes. Abordagem assente no recurso (Resource-Based policies): aplicas via definições IAM ou Bucket Policies as negações ativas perante recursos confinando e fixando que "nesta base purificada a Bucket A apenas o Utilizador Y atua e acede ativamente no S3". Abordagem de Identidade estrita (Identity-Based policies puras): atrelas a uma role de computação "IAM AssumeRoles / ServiceAccounts" da instância Cloud EMR e Spark apenas atuações imperativas `GET` e cirúrgicas direcionadas nas URIs delimitadas ou nos "Tags de atributos dinâmicos exatos" `kinesis:List` do recurso originário, barrando-lhe universalmente qualquer permissão estrita paralela nos restantes nós orquestradores nas infraestruturas em Cloud partilhadas de base originária pura da organização.

## Prós e Contras
- **Prós:** 
  - Tranca brutalmente estrito no imediato e organicamente debaixo dos tetos limitativos imperativos de nuvens qualquer anomalia perigosa de wipeouts acidentais, bloqueando com permissividade de 0-Trust na conta a desastres iminentes perante APIs mal configuradas de destruição.
  - Delega perfeitamente a fiabilidade de acessos aos provedores hiper seguros, geridos via código unificado purificado na Terraform estrita, abstraindo códigos de aplicação de lógicas perigosas no limite base orquestradora.
- **Contras:** 
  - Gera um fardo gigante e muitas dores de manutenção na gestão colossal dos limites. Confinar com o "Book rules" demasiados papéis origina caos nas aprovações base e lentidão orgânica orquestrada (e recorrer por pragmatismo puro a Wildcards exatos e perigosos "prefixos*" acaba ativamente a anular e deturpar a promessa original do padrão purificado legal da framework de acessos).
  - É fortemente travado pelas lógicas de quotas subjacentes nativas na conta cloud originária e purificada (números puros limitados de Policies e IAMs disponíveis nas restrições AWS e GCP).