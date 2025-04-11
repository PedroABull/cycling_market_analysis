[![author](https://img.shields.io/badge/Author-PedroBull-red.svg)](https://www.linkedin.com/in/pedro-bull-0363ba1a1/)
[![](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)

# Inteligência Comercial no Mercado de Ciclismo: Análise de Vendas e Clientes

Este projeto foi desenvolvido como trabalho final do módulo de Engenharia de Dados da Pós-graduação em Data Science e Advanced Analytics da PUC-RJ.

<p align="center">
  <img src="images/wallpaper.png" alt="cover_img" width="900"/>
</p>

<p align="center">
  <strong>Link original para o dataset: https://www.kaggle.com/datasets/sadiqshah/bike-sales-in-europe</strong>
</p>

<p align="center"> 
  <a href="https://www.linkedin.com/in/pedro-bull-0363ba1a1/" target="_blank"><img src="https://img.shields.io/badge/-LinkedIn-%230077B5?style=for-the-badge&logo=linkedin&logoColor=white" target="_blank"></a> 
</p>

## Objetivos

Este projeto tem como objetivo simular uma análise de dados de vendas de uma empresa fictícia do setor de ciclismo, aplicando conceitos de Engenharia de Dados, Processos de ETL e Análises de dados com SQL e Python através da plataforma de dados unificada na nuvem Databricks. A partir da construção de um data warehouse com múltiplas camadas (bronze, silver e gold), buscou-se responder a perguntas estratégicas sobre o comportamento de compra dos clientes, desempenho de produtos, lucratividade por categoria e oportunidades de expansão de mercado.

Perguntas a serem respondidas:
  - Quais tipos de produtos e/ou categoria movimentam mais o mercado?
  - Como o comportamento de compra varia entre diferentes perfis de clientes?
  - Em quais tipos de produtos a empresa deve investir?
  - Quais países representam os maiores mercados para a empresa?

## Estrutura do repositório

O repositório está estruturado da seguinte forma:

```
├── data
├── images
├── notebooks
├── reports
```

- Na pasta `data` a base de dados utilizada no projeto, a qual foi carregada no Databricks através do DBFS(Databricks FIle System)
- Na pasta `images` estão as imagens utilizadas neste README.
- Na pasta `notebooks` está o notebooks com o desenvolvimento do projeto. Em detalhes, temos:
- Na pasta `reports` estão os relatórios gerados durante o projeto, como catálogo de dados e diagramas de modelagem de dados.

## Detalhes do dataset utilizado

### Catálogo de Dados

| Coluna               | Tipo de Dado        | Descrição                                                                | Valores/Intervalos Esperados                                  |
|----------------------|---------------------|--------------------------------------------------------------------------|---------------------------------------------------------------|
| `Sale_Date`          | DATE (yyyy-MM-dd)   | Data completa da venda                                                   | Dentro do intervalo de anos válidos; sem datas futuras        |
| `Day`                | INT                 | Dia do mês em que a venda ocorreu                                        | 1 a 31                                                        |
| `Month_Name`         | STRING              | Nome do mês da venda                                                     | "January" a "December"                                        |
| `Year`               | INT                 | Ano da venda                                                             | Ex: 2011 a 2016                                               |
| `Customer_Age`       | INT                 | Idade do cliente no momento da compra                                    | 10 a 100 (valores fora disso podem indicar erro)              |
| `Customer_Gender`    | STRING              | Gênero do cliente                                                        | "M" para homens e "F" para mulheres                           |
| `Country`            | STRING              | País onde a venda foi realizada                                          | Lista limitada (ex: "United States", "France", "Germany"...)  |
| `State`              | STRING              | Estado ou província da venda                                             | Compatível com o país correspondente                          |
| `Product_Category`   | STRING              | Categoria do produto vendido                                             | "Bikes", "Clothing", "Accessories"                            |
| `Sub_Category`       | STRING              | Subcategoria do produto                                                  | Ex: "Mountain Bikes", "Socks", "Helmets"                      |
| `Product`            | STRING              | Nome do produto                                                          | Nome único por item vendido                                   |
| `Order_Quantity`     | INT                 | Quantidade de unidades vendidas                                          | ≥ 1 (quantidade positiva e plausível)                         |
| `Unit_Cost`          | FLOAT / DECIMAL     | Custo unitário do produto                                                | > 0                                                           |
| `Unit_Price`         | FLOAT / DECIMAL     | Preço de venda unitário                                                  | > 0                                                           |
| `Profit`             | FLOAT / DECIMAL     | Lucro total da venda                                                     | Pode ser positivo ou negativo                                 |
| `Cost`               | FLOAT / DECIMAL     | Custo total da venda (`Unit_Cost` * `Order_Quantity`)                    | > 0                                                           |
| `Revenue`            | FLOAT / DECIMAL     | Receita total da venda (`Unit_Price` * `Order_Quantity`)                 | ≥ 0                                                           |
| `ID_Product`         | BIGINT              | Identificador único do produto (chave primária em DimProducts)           | Autoincremento, valor único                                   |
| `ID_Customer`        | BIGINT              | Identificador único do cliente (chave primária em DimCustomers)          | Autoincremento, valor único                                   |
| `ID_Region`          | BIGINT              | Identificador único da região (chave primária em DimRegion)              | Autoincremento, valor único                                   |
| `ID_Region_Customer` | BIGINT              | Identificador único da combinação região-cliente                         | Autoincremento, valor único                                   |


## Modelagem de dados

O modelo de dados definido para o problema foi o **SnowFlake**. Essa escolha se deu principalmente para evitar a repetição desnecessária de dados e manter a integridade, facilidade de manutenção e a escalabilidade, pensando que esses dados tendem a crescer com o tempo, tanto em dados históricos mas também em número de produtos e regiões.

O primeiro passo foi desenvolver a modelagem conceitual dos dados definindo a dinâmica de relacionamento das diferentes informações contidas na base de dados original. Para esse caso em específico, as informações de clientes que realizaram compra encontram-se agrupadas, ou seja, a granularidade máxima desses atributos é dada como grupos de clientes por idade e por sexo. Sendo assim, uma cidade pode conter 1 ou n grupos de clientes, assim como um grupo de clientes pode estar em uma ou n cidades. Por esse motivo, foi adotada uma abordagem alternativa ao modelo Snowflake tradicional, com a introdução da tabela intermediária **DimRegion_Customers**, que permite manter a consistência dos relacionamentos e garantir flexibilidade para análises futuras.

![boxplot](images/cv_results.png)

O segundo passo foi eliminar os relacionamentos **n:n** do modelo, incluindo a tabela fato de vendas e uma tabela de relacionamento entre as entidades de Região e Grupo_Cliente, conforme descrito na etapa anterior.

![boxplot](images/cv_results.png)

O terceiro e último passo foi a modelagem lógica das entidades e relacionamentos, onde foram definidas quais as colunas devem estar contidas em cada tabela e quais são as chaves primárias e extrangeiras para cruzar as informações.

![boxplot](images/cv_results.png)

Por fim, foram definidas as seguintas tabelas para a camada gold do Data Warehouse:
  - **FactSales**: Tabela fato de vendas 
  - **DimProducts**: Tabela dimensão de produtos
  - **DimCustomers**: Tabela dimensão de clientes
  - **DimRegion**: Tabela dimensão de regiões de venda
  - **DimRegion_Customer**: Tabela contendo todas as combinações de ID_Region e ID_Customer, chaves primárias das tabelas **DimRegion** e **DimCustomers**, respectivamente.

# Modelo Proposto e resultados

A partir de todas as análises realizadas até aqui, foram definidos os seguintes modelos para o problema:

-  Linear Regression
-  Regressão L1(Regularização Lasso)
-  Regressão L2(Regularização Ridge)
-  Decision Tree
-  KNN
-  SVR

Como métricas para avaliação de performance, foram escolhidas:

- R²: Para avaliar o quanto cada modelo se ajusta aos dados disponíveis. Essa métrica foi escolhida com o objetivo de dar suporte às análises.

- RMSE: Para avaliar, em escala, qual a magnitude média do erro de previsão de cada modelo. Essa é a métrica principal para seleção do modelo justamente pelo fato de fornecer uma referência média de quanto o modelo está errando em relação ao valor real.

Os resultados obtidos para os primeiros testes estão detalhados no gráfico abaixo:

![boxplot](images/cv_results.png)

A princípio, boa parte dos modelos performou relativamente bem nos dados de treino, com exceção da Regressão L1.

Em seguida, foi realizada uma etapa de otimização de hiperparÂmetros com o GridSearch para extrair o melhor de cada um dos modelos propostos. O resultado final segue detalhado abaixo:

![results](images/results.png)

Resultados alcançados:

1. É perceptível que a dispersão dos pontos do modelo SVR, em relação à reta de referência, é considerávelmente menor do que os demais modelos.

2. Com o maior volume de dados disponíveis e, também, com a otimização dos hiperparâmetros, a regressão L1 se aproximou bastante da regressão L2, tendo o RMSE reduzido em mais de 50%, chegando a menos de 3 mpg na fase final de testes.

3. Antes da etapa de otimização, o algotimo KNN era o que estava performando melhor, com um RMSE de 2.30 mpg. Após a otimização, o algoritmo SVR melhorou sua performance consideravelmente, apesar de demorar o dobro do tempo de treinamento, em média, em relação ao KNN para processar os dados, como mostra a tabela com os resultados da validação cruzada. Ainda assim, todos os modelos processaram os dados com bastante celeridade.

# Conclusão

Ao final, o projeto alcançou resultados satisfatórios em relação aos tempos de processamento dos dados e, principalmente, para os resultados alcançados.

O modelo final escolhido para a resolução do problema foi o modelo SVR, o qual teve uma performance cerca de 8% melhor do que o segundo melhor modelo testado. O modelo SVR foi tanto o que se ajustou melhor aos dados de maneira geral, com um R² de 0.93, quanto o que teve o menor valor de erro, com um RMSE de 1.97 mpg.
