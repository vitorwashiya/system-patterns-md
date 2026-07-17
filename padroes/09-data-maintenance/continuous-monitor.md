# Padrão: Monitor Contínuo (Continuous Monitor)

## 1. Resumo (O que é?)
Ao invés de barrar a execução como o *Quality Checker*, o Monitor Contínuo apenas inspeciona, levanta estatísticas sistêmicas do dado a longo prazo (Data Observability) e alerta os mantenedores quando os padrões fogem de uma distribuição normal preestabelecida, sem impedir a gravação dos dados nas tabelas finais.

## 2. O Problema
* Seu *Quality Checker* se tornou um fardo. Você impôs uma regra de "Receita diária deve ser > 10.000". Num domingo de feriado, a receita foi de apenas 5.000. O Verificador de Qualidade achou que era um erro de ingestão e quebrou o pipeline inteiro.
* A diretoria ficou furiosa porque os relatórios da segunda-feira atrasaram inteiros só porque o sistema não tolerou a anomalia normal do feriado comercial.

## 3. A Solução
Delegar regras subjetivas para o Monitor Contínuo. Este padrão permite que a carga sempre seja concretizada. O pipeline flui livre. Paralelamente (como um "sidecar" ou serviço secundário), o monitor compila metadados sobre o lote gravado (Ex: Quantidade de registros, Média de receita, Porcentagem de nulos). Ele envia essas amostras para uma tabela ou serviço central (Data Dog, Monte Carlo, Soda). Se os painéis detectarem uma deriva bizarra (Data Drift), eles apitam nos canais do Slack/Teams da equipe de engenharia para que façam uma auditoria humana, mas o serviço para o negócio não parou de rodar.

## 4. Consequências e Trade-offs
* **Vantagens:** Abordagem pacífica sem "falsos positivos restritivos". Protege SLAs comerciais severos que não podem se dar ao luxo da pipeline abortar na nuvem, garantindo entrega (ainda que suspeita) ao usuário.
* **Desvantagens/Atenção:** 
  * **O Poluente já Entrou:** O monitor só avisa *depois* que os fatos ocorreram. Se a receita não foi 5 mil por causa do feriado, mas sim porque o banco truncou o último zero dos valores reais na ingestão, o dado podre JÁ ESTÁ nas tabelas principais. Os analistas trabalharão com lixo até o engenheiro ler a notificação, e o backfilling manual será inevitável.

## 5. Exemplo de Aplicação Prática
Uma tabela transacional de cliques sobe dados de segundo a segundo sem barreiras para uma tabela no Redshift. Um software de Observabilidade varre os metadados dessa tabela a cada 4 horas no Redshift. Se ele perceber que a quantidade habitual de cliques caiu 40% em relação aos desvios-padrões das quartas-feiras normais, lança um alarme vermelho no PagerDuty do Engenheiro de Plantão para averiguar se a página quebrou.

## 6. Exemplo Simples de Código
```python
# Em vez de Raise Exception, o monitor grava estatísticas livremente 
def registrar_metricas_contínuas(df):
    quantidade_nulos = df.filter(col('email').isNull()).count()
    total = df.count()
    porcentagem = (quantidade_nulos / total) * 100
    
    # Grava na Tabela de Governança ou Datadog para painéis avisarem anomalia
    enviar_metrica_para_dashboard("vendas_taxa_de_nulos_no_email", porcentagem)

# O código continua rodando e salva a base, não importa a porcentagem
df.write.save("base_publica_vendas")
```

## 7. Padrões Relacionados ou Nomes Similares
Pertencente vital às disciplinas de *Data Observability*. Pode operar de forma desacoplada ou como um *Isolated Monitor*.
