# Vertical Partitioner (Data Removal)

**Categoria:** Segurança de Dados

## 🎯 Objetivo
Organizar topologicamente bases purificando a repetitividade da informação estática em dados estritos de identidade legal de um único ponto para facilitar agilmente eliminações ou "Right to be Forgotten" exigido pelas legislações puras (GDPR, CCPA).

## O Problema
Tens dados brutos com colunas e instâncias repetidas em todas as milhões de visitas estritas ao portal. Como as normas te impõem o encargo estrito da eliminação pessoal tens de assegurar que purgas identidades, datas originais, CPFs do volume colossal de forma ágil, minimizando reescritas constantes perante partições ativas de imensas colunas estáticas de identidade replicadas para o mesmo evento de utilizador dezenas ou centenas de vezes iteradas em todo o modelo de dados imenso.

## A Solução
O padrão recorre ao isolamento físico restritivo do modelo de "Vertical Partitioner" no ato das purificações. Basicamente em contraposição com cortes horizontais puros a lógica fatia as colunas extraídas imutáveis numa base pura segregada estrita. Por dedução orgânica do Spark de escrita, filtra os "IDs e PII contextuais fixos" do "event" em separado na Tabela Mestre da Ingestão estrita. Ao apagar dados a jusante ou invocar o apagamento das regras legais em invocações (VACUUM puro), basta destruir nativamente e individualmente num ponto orgânico estrito único da base central pequena de PII minimizando IO restritivo na procura e alteração dos pesados dados brutos ativos das estatísticas no lado relacional bruto de grandes blocos do evento.

## Prós e Contras
- **Prós:** 
  - Reduz fenomenalmente os custos operacionais (IO Overhead) das restrições puras das operações massivas base da cloud para purgas, facilitando enormemente o foco da eliminação isolada legal.
  - Oferece retenções de políticas puramente autónomas onde eliminas ativamente identidades mas podes conservar para sempre agregados não referenciáveis ativos em pastas isoladas desvinculadas das restrições na Cloud original transacionada da plataforma analítica.
- **Contras:** 
  - Empurra inexoravelmente a complexidade sistémica na arquitetura de forma agressiva (Domain Split) requerendo que leitores montem ou invoquem `JOINs` massivos estritos via rede perante partições para re-obterem os contextos das colunas originais na tabela.
  - Altamente complexo no modelo do "Polyglot" em arquiteturas multi-base (e.g. motores rápidos NoSQL operacionais) exigindo infraestruturas com canais paralelos ou streams massivas para desdobramentos operacionais nas escritas dos múltiplos tópicos sincronizados no log inicial.