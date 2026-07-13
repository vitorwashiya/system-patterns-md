# Dynamic Joiner

**Categoria:** Valor dos Dados

## 🎯 Objetivo
Interligar bases vivas provenientes de sistemas orientados ao streaming e de alta flutuação temporal superando o perigo clássico de desfasamentos das atualizações originadas pelas componentes latentes presentes nas infraestruturas reativas assíncronas do Big Data.

## O Problema
Tentaste o modelo anterior mas tens de atuar em infraestrutura pura de stream com PII changes oriundos da frente via "Change Data Capture". Devido à assincronicidade pesada inerente a ambientes deste género tens perdas brutais de correlações e os teus modelos analíticos ignoram correlações reais porque a sincronização de eventos que se espera do "User-Clicks" surge mais cedo nas filas e cruza no vazio com a do "User-Profile-Updates" recém originada noutra cauda que fica mais estancada no meio do fluxo temporário.

## A Solução
Como o "Static Joiner" é incapaz de agir eficaz e aguardar nestas realidades em paralelo e vivo, usa-se a semântica reativa imposta pelo padrão Dynamic Joiner. A arquitetura instiga ativamente limites base e limites alargados (time boundaries). Na prática estipula a configuração contínua dos buffers alocados temporariamente. Retêm informações e os eventos recém formados num dos córregos pelo intervalo tolerante que permita as linhas divergentes em desfasamento da ponta adversa equipararem, ou seja, intercetarem os dados atrasados à justa antes de se processar um match forçado de janelas.

## Prós e Contras
- **Prós:** 
  - Mitiga a perda analítica nos cruamentos das topologias onde a latência oscila. Garante um tratamento fiel de semânticas reais transpostas na vivacidade paralela.
  - Assemelha-se fortemente a soluções implementadas via bibliotecas de suporte nativo (Flinks via joins com tempo temporal (Temporal table joins)).
- **Contras:** 
  - Aumenta absurdamente as restrições a nível computacional (trade-off do "Espaço versus Exactidão") na orquestração devido a ter as máquinas a reter permanentemente listas e coleções longas nos stores de memória interna para albergarem limites em compassos de GC watermarks.
  - Não resolve a totalidade das intersecções perdidas se por inércia fatal as ligações quebrarem originando falhas que extrapolam radicalmente o alcance e teto permissível da tolerância temporal designada.