# Padrão: Compactador (Compactor)

## 1. Resumo (O que é?)
O padrão Compactador foca na redução da pegada de armazenamento unindo vários arquivos pequenos em arquivos maiores. Na engenharia de dados, ler muitos arquivos pequenos é extremamente custoso; portanto, este padrão atua otimizando o processo de leitura para os consumidores a jusante.

## 2. O Problema
* O seu pipeline de ingestão em tempo real cria arquivos pequenos de poucos kilobytes a cada minuto em um storage (Data Lake/S3).
* Com o tempo, trabalhos em lote (batch) começam a levar muito tempo apenas para listar e abrir esses milhares de pequenos arquivos (o clássico "Problema de Arquivos Pequenos"), encarecendo os serviços na nuvem e reduzindo a performance.

## 3. A Solução
Aplicar o padrão Compactador significa executar um processo (geralmente em background ou com uma frequência menor) que lê lotes de pequenos arquivos e os agrupa e reescreve em arquivos bem maiores e otimizados (como um Parquet de 128 MB a 1 GB). Ferramentas modernas de formatos abertos (como Delta Lake, Apache Iceberg) costumam possuir comandos nativos (como `OPTIMIZE` ou `REWRITE DATA FILES`) que fazem isso com eficiência transacional.

## 4. Consequências e Trade-offs
* **Vantagens:** Melhora absurdamente o tempo de leitura de dados; diminui o overhead de I/O de rede e chamadas de API (como listagens no S3).
* **Desvantagens/Atenção:** 
  * **Custo vs Performance:** A compactação em si é um processo que gasta recursos de processamento (computação intensa). É preciso achar o equilíbrio para não rodá-la com muita frequência (gasto excessivo) nem com pouca (deixando os consumidores sofrerem com leitura lenta).
  * **Necessidade de Limpeza (Vacuum):** Os arquivos originais (pequenos) podem não ser removidos automaticamente logo após a compactação. É necessário rodar processos de limpeza física (housekeeping/vacuum) frequentemente para não duplicar o custo de armazenamento.

## 5. Exemplo de Aplicação Prática
Seus eventos de IoT chegam de milhares de sensores por segundo. Um processo rápido salva tudo no storage a cada 1 minuto (gerando milhares de mini-arquivos). Todo dia à meia-noite, um script de Compactação "varre" os arquivos daquele dia e os junta em apenas 5 ou 6 arquivos grandes e compactados, facilitando a consulta dos cientistas de dados no dia seguinte.

## 6. Exemplo Simples de Código
```python
# Em Delta Lake, você não precisa fazer a lógica na mão,
# apenas invocar o comando de otimização nativo
tabela_dispositivos = DeltaTable.forPath(spark_session, diretorio_tabela)
tabela_dispositivos.optimize().executeCompaction()
```

## 7. Padrões Relacionados ou Nomes Similares
Frequentemente associado ao *Vacuum/Housekeeping* (para remoção de arquivos não utilizados). Tenta resolver deficiências de performance e custo não resolvidas na *Carga Incremental* (Incremental Loader) ou *Change Data Capture*.
