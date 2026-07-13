# External Trigger

**Categoria:** Ingestão de Dados

## 🎯 Objetivo
Acionar instâncias e processos do pipeline não com tempos fixos no relógio da orquestração, mas com "push notifications" assíncronas geradas sempre que novos pacotes de dados ganham forma, de origem em ações dos produtores externos de base.

## O Problema
As execuções calendarizadas tradicionais ativam processos com bastante desperdício; como a chegada de dados por um fornecedor parceiro ocorre aleatoriamente de segunda a quinta-feira sem período previsto de envio, agendar à sorte consome computação ou causa atrasos entre o momento da chegada da informação útil e o agendamento natural subsequente do cron da infraestrutura. Queremos focar os nossos orçamentos num perfil orientado à ativação em cadeia.

## A Solução
Este modelo baseia a invocação na presença do evento propriamente dito. O componente produtor "empurra" avisos de envio/término num message broker comum que serve de infraestrutura nervosa da arquitetura de fluxo (ex: funções lambda despoletadas numa bucket do S3 ou um Kafka broker subscrito). Assim que esta infraestrutura processadora escuta os ecos, emite comandos no sistema orquestrador que começa ativamente a processar instantaneamente e com segurança os recursos partilhados do pacote referenciado no objeto.

## Prós e Contras
- **Prós:** 
  - Minimiza em grande medida os atrasos operacionais por inação e os grandes e ociosos tempos mortos do agrupamento sequencial por horário.
  - Otimiza o rácio entre despesas do servidor com execuções redundantes diárias em lote.
- **Contras:** 
  - Requer fortes mecanismos de recuperação face ao desastre. Perdas da mensagem do broker invalidam para o resto dos dias a ativação dessa tarefa.
  - Um agendamento puramente mecânico deste cariz retira bastante inteligibilidade contextual da pipeline e dificulta ao engenheiro ter conhecimento nativo do que acionou do fluxo em causa.