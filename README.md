# Churn Analysis
Repositório: https://github.com/wewerthonc/churn-analysis

Original: https://colab.research.google.com/drive/1HMHLvDkqChgJUFPj-5Iw7-vpHcfca8ud.
- Google colab notebook: **Churn.ipynb**
- Dataset: data\data-test-analytics.csv

Neste notebook, montamos o Google Drive, importamos pacotes necessários como matplotlib, seaborn, numpy e pandas, e lemos um arquivo CSV chamado "data-test-analytics.csv", que contém dados de churn de clientes.

# 1. Introduction

Estou participando de um desafio para uma vaga de estágio. A empresa em questão, especializada em serviços de assinaturas, está enfrentando um problema de aumento no índice de "Churn", ou seja, clientes que assinam o serviço e o cancelam em algum momento posterior. A equipe de assinaturas tem como objetivo reduzir essa perda de assinantes, porém, mesmo com melhorias na plataforma, o churn vem aumentando nos últimos meses. Nesse contexto, será realizada uma análise de dados com o objetivo de identificar os fatores que levam à perda de assinantes e propor soluções o problema.

# 2. Data Description
Como dito previamente, o dataset contém informações de assinaturas de clientes dessa empresa. Cada linha do dataset representa um cliente e este possui 10000 linhas. Cada coluna significa uma característica do cliente, sendo as seguintes colunas:
- id: Identificação do cliente
- created_at: Data de criação da assinatura
- updated_at: Data da última modificação da assinatura
- deleted_at: Data de cancelamento da assinatura
- name_hash: Nome do usuário criptografado
- email_hash: E-mal do usuário criptografado
- address_hash: Endereço do usuário criptografado
- birth_date: Data de aniversário do cliente
- status: Status da assinatura
- version: Versão da assinatura
- city: Cidade do cliente
- state: Estado do cliente
- neighborhood: Bairro do cliente
- last_date_purchase: Data do último pedido que ocorreu pela assinatura
- average_ticket: Média de gasto por pedido
- items_quantity: Média de itens na assinatura
- all_revenue: Total de receita realizado pelo cliente
- all_orders: Total de pedidos realizado pelo cliente
- recency: Tempo desde a última compra do cliente
- marketing_source: Canal de marketing que converteu a assinatura

<p align="justify">
Serão usadas todas as colunas para a análise, com exceção das colunas name_hash, email_hash e address_hash, visto que não são importantes para análise.
</p>

<p align="justify">
Logo no início, é necessário definir as colunas que contêm dados do tipo data, pois pode ajudar a evitar erros e aumentar a precisão das análises.
</p>

```python
# Define the column names to convert to date
date_cols = ['created_at', 'updated_at', 'deleted_at', 'birth_date', 'last_date_purchase']

# Read the CSV file and convert specified columns to datetime
churn_df = pd.read_csv(PATH_BASE + "data-test-analytics.csv", usecols = cols, parse_dates = date_cols)
```
<p align="justify">
Dessa forma, assim ficará o dataset. Note que há valores 'null' para a coluna 'deleted_at'. Obviamente, clientes ativos não contém uma data de cancelamento da assinatura.
</p>

<p align="center">
  <img src= "images/churn_df_info.png" />
</p>

# Data Clearning

<p align="justify">
Um dos primeiros passos na limpeza de dados, é a contagem de valores nulos por coluna. Com essa informação, conhecemos mais sobre o dataset e podemos decidir como manipular os dados. Da mesma forma, ajuda a evitar erros nas análises que virão.
</p>

<p align="center">
  <img src= "images/null_values.png" />
</p>

#### Data out of range

<p align="justify">
Nas análises a serem realizadas, é importante que tenhamos valores consistentes. No mundo real, é impossível alguém nascer no futuro, assim como alguém não pode comprar uma quantidade negativa de itens. O código abaixo seleciona as colunas que contêm datas e as colunas numéricas. Para as colunas com datas, é verificado a quantidade de valores no futuro e para as colunas numéricas e verificado a quantidade de valores negativos.
</p>

```python
from datetime import date

# Check for out of range values in datetime columns
date_cols = churn_df.select_dtypes(include=['datetime64']).columns
for col in date_cols:
    num_out_of_range = len(churn_df[churn_df[col].dt.date > date.today()])
    print(f"Number of '{col}' rows out of range: {num_out_of_range}")

# Check for negative values in numeric columns
num_cols = churn_df.select_dtypes(include=['int64', 'float64']).columns
for col in num_cols:
    num_negative = len(churn_df[churn_df[col] < 0])
    print(f"Number of '{col}' rows out of range: {num_negative}")
```

<p align="center">
  <img src= "images/out_of_range_data.png" />
</p>

<p align="justify">
Por meio dos resultados, é possível perceber bastante valores incorretos para a coluna 'birth_date'. A quantidade representa mais de 50% dos valores contidos na tabela. Sendo assim, é importante excluir essa coluna do dataset, uma vez que não irá nos ajudar a tirar conclusões sobre o problema.
</p>

```python
# Drop 'birth_date' column
churn_df.drop('birth_date', axis = 1, inplace = True)
```

#### Duplicated Data

<p align="justify">
Outra verificação muito importante quando estamos limpando os dados, é a contagem de valores duplicados. Se tivessemos o mesmo cliente duplicado centenas de vezes no nosso dataset, poderiamos tirar conclusões enviesadas sobre o nosso problema. Portanto, não seria uma situação agradável. Essa verificação pode acontecer de várias formas. Porém, é sempre necessário olharmos para valores duplicados considerando todas as colunas e para a coluna que deveria ter um valor único, muitas vezes chamada de chave primária.
</p>

```python
# Drop duplicates across all columns
churn_df.drop_duplicates(inplace = True)

# Drop duplicated rows that have the same id
churn_df.drop_duplicates('id', inplace = True)
```

#### Inconsistent Data

<p align="justify">
Pode haver diversas inconsistências nos nossos dados. Muitas vezes, os dados podem estar corrompidos, incompletos ou imprecisos. Essas inconsistências podem afetar a qualidade da análise e gerar resultados errados ou distorcidos. No nosso dataset, podemos observar que temos várias colunas representado datas. Com base nessas datas, podemos chegar a seguinte afirmação: um dado da coluna 'created_at' não deve ser maior que um dado da coluna 'updated_at', assim como não deve ser maior que um dado da coluna 'deleted_at'. Um cliente não pode ser criado depois de deletado ou atualizado. Da mesma forma, um dado da coluna 'last_date_purchase' não pode estar no futuro, ou ter acontecido depois do cliente ter sido deletado.
</p>

```python
# Define a mask to filter the rows based on the following conditions:
# 1. 'created_at' should be less than or equal to 'updated_at'
# 2. 'created_at' should be less than or equal to 'deleted_at'
# 3. 'created_at' should be less than or equal to 'last_date_purchase'
# 4. 'last_date_purchase' should be less than or equal to 'deleted_at' (if it exists)
#    Otherwise, it should be less than or equal to the current date and time

mask = (churn_df['created_at'] <= churn_df['updated_at']) & \
        (churn_df['created_at'] <= churn_df['deleted_at'].fillna(pd.Timestamp.now())) & \
        (churn_df['last_date_purchase'] <= churn_df['deleted_at'].fillna(pd.Timestamp.now())) & \
        (churn_df['created_at'] <= churn_df['last_date_purchase'])

# Filter the churn data using the mask and update the churn data with the filtered data
consistent = churn_df[mask]
inconcistent = churn_df[~mask]
```

<p align="justify">
Ao todo foram encontrados 177 valores inconsistentes. Felizmente, é um número pequeno. Agora, devemos excluir os clientes com esses valores ou atualiza-los?! O que aconteceria se todos esses valores inconsistentes fossem de clientes que cancelaram a assinatura?! Isso tornaria a nossa análise muito mais difícil. Visto que, como visto anteriormente, somente 505 possuem possuem data de cancelamento. Perderiamos a representação desses clientes. Por isso, é importante analisar quais são as classes contidas nesses dados inconsistentes. Por meio da coluna 'status', é possível ver essa informação.
</p>

<p align="center">
  <img src= "images/inconsistent_status.png" />
</p>

<p align="justify">
Como nenhum dos clientes são clientes que cancelaram a assinatura, podemos retirá-los do dataset sem perda de informação.
</p>

#### Cross-validation

<p align="justify">
Não, não estamos falando de machine learning. Cross-validation, no contexto de limpeza de dados, significa juntar informações de algumas colunas para verificar a consistência do valor de outra coluna. No nosso dataset, por meio da colunas 'average_ticket' e 'all_orders', é possível conseguir o valor da coluna 'all_revenue'. Assim como, com os valores da coluna 'deteled_at', é possível saber os clientes com status de cancelado na coluna 'status'.
</p>

```python
# Check if avarage ticket times all_orders is equal to all_revenue
# Take precision when multiplicating floats into account
tolerance = 0.1
mask = abs(churn_df['average_ticket'] * churn_df['all_orders'] - churn_df['all_revenue']) < tolerance
churn_df = churn_df[mask]
```

```python
# Select rows where 'deleted_at' column is not null
deleted_churn = churn_df[~pd.isna(churn_df['deleted_at'])]

# Select rows where 'status' column is 'canceled'
canceled_churn = churn_df[churn_df['status'] == 'canceled']

# Check if all canceled subscriptions have 'deleted_at' values
result = (deleted_churn == canceled_churn).isna().any().sum() == 0

if result:
  print("All canceled subscriptions have 'deleted_at' values")
```

<p align="justify">
A coluna 'recency' possui valores que representam o tempo desde a última compra do cliente. O que aconteceria se esses valores continuassem sendo alterados mesmo depois do cliente ter cancelado a assinatura ?! Caso comparássemos esses valores com clientes ativos, poderíamos chegar a falsas conclusões, tal como alegar que um cliente que fica muito tempo sem comprar com a assinatura, tem tendência a cancela-la. Dessa forma, vamos criar uma nova coluna 'recency_subscription', que significa a quantidade de dias que um cliente está sem fazer uma compra, levando em consideração somente o tempo de assinatura. A data da compra mais nova no dataset será usada como referência para fazer os cálculos para clientes ativos.
</p>

```python
# Create a new column that shows the time since the last purchase in the subscription
churn_df['recency_subscription'] = (churn_df['last_date_purchase'].max() - churn_df['last_date_purchase']).dt.days


# For clients who have a value in the 'deleted_at' column, calculate the difference
# between the 'deleted_at' and 'last_date_purchase' columns and use that value to
# update the coolumn 'recency_subscription'.
churn_df.loc[churn_df['deleted_at'].notnull(), 'recency_subscription'] = \
          (churn_df['deleted_at'] - churn_df['last_date_purchase']).dt.days
```

# Exploratory Data Analysis

# Conclusion
