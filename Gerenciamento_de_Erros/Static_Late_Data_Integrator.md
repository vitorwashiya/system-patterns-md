# Static Late Data Integrator

**Categoria:** Gerenciamento de Erros

## 🎯 Objetivo
Integrar e reprocessar os valiosos dados que chegaram "atrasados", definindo um ciclo retrospectivo estático, reajustando automaticamente e de forma transparente partições do passado durante execuções comuns da pipeline.

## O Problema
Na tua análise temporal o "Late Data Detector" expôs-te que muitas conexões falharam mas que ignorar o que demorou a chegar seria um desperdício doloroso: estatísticas estariam perenemente erradas. Tens dados limitados por 15 dias de atraso (um SLA permissível) e não podes correr 15 scripts em separado diariamente de forma manual para consolidá-los.

## A Solução
Usa o Static Late Data Integrator implementando um período retrospectivo fixo (Static Lookback Window). Todos os dias, a tua pipeline (além de calcular dados para "hoje") efetua backfill ou recomputa sistematicamente partições correspondentes aos N dias passados previstos. Este agendamento, embora cego a se houve verdadeiramente alterações no passado ou não, funde os dados atrasados na respetiva posição e restabelece a realidade histórica da partição.

## Prós e Contras
- **Prós:** 
  - Extremamente fácil de programar; apenas usa um loop de tempo retrospectivo sem necessitar de verificar estados internos.
  - Consistência analítica total para eventuais sistemas e parceiros a que forneces a base.
- **Contras:** 
  - Custo operacional considerável: a máquina efetua constantemente processamentos "em vazio" reescrevendo/analisando informação nos dias antigos mesmo que os dados atrasados sejam inexistentes nesse instante.
  - Pode desencadear o efeito perverso da "bola de neve do backfill", onde cada atualização que forças aos 15 dias passados obriga equipas a montante a recomputar tudo de igual modo.