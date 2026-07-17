# Padrão: Sobrescritor No Local (In-Place Overwriter)

## 1. Resumo (O que é?)
O Sobrescritor No Local (*In-Place Overwriter*) aborda a exclusão de dados buscando e substituindo ou destruindo fisicamente linhas isoladas ou blocos no próprio conjunto original. Ele age retroativamente no dado quando a separação cautelosa prévia (como o *Vertical Partitioner*) não foi feita ou não está disponível.

## 2. O Problema
* Você herdou um lago de dados legado com Petabytes particionados por hora. Os dados sensíveis PII estão espalhados em todas as colunas.
* Uma solicitação da agência regulatória obriga a exclusão dos dados de um usuário sob demanda, mas o design atual não permite desvincular a identidade apagando uma tabela central menor.

## 3. A Solução
Como não há como desviar do arquivo físico, você invoca o padrão de exclusão nativa (ex: `DELETE FROM tabela WHERE usuario='X'`). Em formatos abertos modernos (Iceberg, Delta Lake), o motor lida com essa operação varrendo os metadados para achar qual arquivo o usuário 'X' está dentro, lendo o arquivo que contém esse lixo, apagando as linhas marcadas e re-escrevendo um arquivo novo limpo no local. 

Para sistemas de dados crús (arquivos puros JSON/CSV), onde o DML não existe, a "Sobrescrita no Local" precisa ser orquestrada na unha (via script Python): o script lê tudo, filtra o usuário X para fora, e salva numa pasta "Staging". Depois de salvo, copia da "Staging" sobrescrevendo brutalmente a "Pasta Principal".

## 4. Consequências e Trade-offs
* **Vantagens:** Abordagem universal aplicável a qualquer arquitetura sem ter que refazer todo o modelo de ingestão retrospectivamente.
* **Desvantagens/Atenção:** 
  * **Custo Exorbitante do I/O e Lentidão:** O banco lê milhões de linhas de um bloco apenas para apagar a linha de um usuário específico, regravando todas as outras linhas no processo. 
  * **Retenção Desnecessária:** Se você deleta uma linha e ela sai da tabela, o arquivo físico velho com a linha (arquivos mortos) continuará retido por dezenas de dias nos servidores garantindo "Time-Travel" (viagem no tempo) do formato Delta/Iceberg, o que, ironicamente, retém legalmente o PII apagado. A operação requer execução imperativa contínua do processo de limpeza real do lixo (`VACUUM`).

## 5. Exemplo de Aplicação Prática
A cada final da semana, a equipe recebe as solicitações da lei de proteção de dados. Na sexta à noite (para não travar análises da semana comercial da equipe de negócios), um job `Spark` com um loop pesado dispara dezenas de operações de DELETE buscando o e-mail de cada requerente ao longo das vastas tabelas do Data Lake e purga o cache.

## 6. Exemplo Simples de Código
```sql
-- Em um Delta Lake 
DELETE FROM clientes_legado WHERE cliente_id = 'ABCD-1234';

-- Exclusão permanente não recuperável removendo arquivos não referenciados pela tabela (lixo de exclusão) retidos nas últimas 0 horas
VACUUM clientes_legado RETAIN 0 HOURS;
```

## 7. Padrões Relacionados ou Nomes Similares
Uma mutação agressiva dos princípios de remoção presentes em *Data Overwrite*. Uma alternativa rudimentar (porém universal) ao *Vertical Partitioner*.
