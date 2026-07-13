# Proxy

**Categoria:** Idempotência

## 🎯 Objetivo
Isolar ativamente os consumidores do ciclo interno em mutação através de entidades intermédias fixas imutáveis, respondendo a normas onde dados nunca podem ter atualizações nem remoções de segurança após inserções no sistema base.

## O Problema
Para aprimorares velhas bases alteraste pipelines iterativas aplicando a sobreposição em tabelas via "Full Loader" e sobrescritas. Surge o departamento legal dizendo que "manutenções do estado antigo por força da legislação ou de compliance obrigam à persistência física dos backups do original". A reescrita na mesma tabela cessa de ser aceite e requeres que a evolução decorra guardando snapshots perpétuos mas apresentando sempre "o que importa" às frentes operacionais atuais, que por seu turno abdicam de estar a mudar caminhos a cada invocação.

## A Solução
Resolvemos isto perante a perspetiva clássica de colocar pontos e indirecionamentos entre níveis subjacentes: usamos o padrão Proxy (Idempotência orientada a caminhos). Este engendra o processo inicial a gravar conjuntos em repositórios fisicamente independentes para ficheiros rotulados via marcas exclusivas (ex. datetime/epoch stamps). Consequentemente, para a comunidade a aceder os dados não fragmentarem conexões em constante descoberta o intermediador aciona, por fim, atualizações nas visualizações dinâmicas, ponteiros estáticos como os Manifest files ou View Abstracts no DW para refletirem apenas as mais recentes alocações inalteráveis originadas.

## Prós e Contras
- **Prós:** 
  - Permite respeitar restrições legais imutáveis tipo WORM (Write Once Read Many) utilizando bloqueios rígidos em objetos depositados em armazenamentos puros do lago de ficheiros.
  - Garante total blindagem das arquiteturas para o consumo dos relatórios já que nenhuma re-escritura avulsa da estrutura causará travões enquanto não for formalmente mudado o apontador Proxy.
- **Contras:** 
  - Nem todos as persistências oferecem suporte exato via ponteiro a coleções múltiplas no repositório final o que força a abordagens morosas baseadas na reescritura total dos pesados ficheiros estáticos extra (Manifest).
  - Perigo agudo perante as alterações imprevistas das topologias (evolutions schema) que requerem atualizações pesadas que afetam todas as interligações fixas subjacentes expostas do intermediário proxy.