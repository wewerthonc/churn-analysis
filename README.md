# Churn Analysis
Repositório: https://github.com/wewerthonc/churn-analysis

Original: https://colab.research.google.com/drive/1HMHLvDkqChgJUFPj-5Iw7-vpHcfca8ud.
- Google colab notebook: **Churn.ipynb**
- Dataset: data\data-test-analytics.csv

Neste notebook, montamos o Google Drive, importamos pacotes necessários como matplotlib, seaborn, numpy e pandas, e lemos um arquivo CSV chamado "data-test-analytics.csv", que contém dados de churn de clientes.

# 1. Introduction

Estou participando de um desafio para uma vaga de estágio. A empresa em questão, especializada em serviços de assinaturas, está enfrentando um problema de aumento no índice de "Churn", ou seja, clientes que assinam o serviço e o cancelam em algum momento posterior. A equipe de assinaturas tem como objetivo reduzir essa perda de assinantes, porém, mesmo com melhorias na plataforma, o churn vem aumentando nos últimos meses. Nesse contexto, será realizada uma análise de dados com o objetivo de identificar os fatores que levam à perda de assinantes e propor soluções o problema.

# 2. Data Description
Como dito previamente, o dataset contém informações de assinaturas de clientes dessa empresa. Ele possui 10000 linhas e contém seguintes colunas:
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

Serão usadas todas as colunas para a análise, com exceção das colunas name_hash, email_hash e address_hash, visto que não são importantes para análise.

Logo no início, é necessário definir as colunas que contêm dados do tipo data, pois pode ajudar a evitar erros e aumentar a precisão das análises.
```python
# Define the column names to convert to date
date_cols = ['created_at', 'updated_at', 'deleted_at', 'birth_date', 'last_date_purchase']

# Read the CSV file and convert specified columns to datetime
churn_df = pd.read_csv(PATH_BASE + "data-test-analytics.csv", usecols = cols, parse_dates = date_cols)
```

<center>
  <img src="imagens/churn_df_info.png">
</center>

