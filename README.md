# Atividade Avaliativa 1 - Tratamento de dados

! pip install pandas

import pandas as pd
import numpy as np

df = pd.read_csv("base_vendas.csv")
df.head()

df.shape

df.info()

df.describe()

print(df.columns)

# 3. Estatísticas simples

print("\nResumo estatístico:")
print(df.describe(include="all"))

print("\nMédia do preço unitário:", df["preco_unitario"].mean())
print("Mediana do preço unitário:", df["preco_unitario"].median())
print("Média da quantidade:", df["quantidade"].mean())

# 4. Qualidade: nulos e duplicados

print("\nValores nulos por coluna:")
print(df.isna().sum())

print("\nRegistros duplicados:", df.duplicated().sum())

# 5. Padronização de categorias

df["cidade"] = df["cidade"].astype("string").str.strip().str.lower()
df["categoria"] = df["categoria"].astype("string").str.strip().str.lower()
df["forma_pagamento"] = df["forma_pagamento"].astype("string").str.strip().str.lower()
df["status"] = df["status"].astype("string").str.strip().str.lower()

df["cidade"] = df["cidade"].replace({
    "são paulo":"São Paulo", "sao paulo":"São Paulo",
    "rio de janeiro":"Rio de Janeiro",
    "belo horizonte":"Belo Horizonte",
    "curitiba":"Curitiba", "porto alegre":"Porto Alegre",
    "salvador":"Salvador", "brasília":"Brasília", "brasilia":"Brasília"
})

df["categoria"] = df["categoria"].replace({
    "eletrônicos":"Eletrônicos", "eletronicos":"Eletrônicos",
    "roupas":"Roupas", "casa":"Casa",
    "esportes":"Esportes", "livros":"Livros"
})

df["forma_pagamento"] = df["forma_pagamento"].replace({
    "cartão":"Cartão", "cartao":"Cartão",
    "pix":"PIX", "boleto":"Boleto", "dinheiro":"Dinheiro"
})

df["status"] = df["status"].replace({
    "concluída":"Concluída", "concluida":"Concluída",
    "cancelada":"Cancelada", "pendente":"Pendente"
})

df.head()

# 6. Tratamento de valores nulos

for col in ["cidade", "categoria", "forma_pagamento"]:
    df[col] = df[col].fillna("Não informado")

df["quantidade"] = pd.to_numeric(df["quantidade"], errors="coerce")
df["preco_unitario"] = pd.to_numeric(df["preco_unitario"], errors="coerce")

# Valores <= 0 são considerados inválidos

df.loc[df["quantidade"] <= 0, "quantidade"] = np.nan
df.loc[df["preco_unitario"] <= 0, "preco_unitario"] = np.nan

# Substituição pela mediana

df["quantidade"] = df["quantidade"].fillna(df["quantidade"].median())
df["preco_unitario"] = df["preco_unitario"].fillna(df["preco_unitario"].median())


df.head()

# 7. Criação de novas colunas

df["valor_total"] = df["quantidade"] * df["preco_unitario"]

df["faixa_venda"] = pd.cut(
    df["valor_total"],
    bins=[-np.inf, 500, 1500, np.inf],
    labels=["Baixa", "Média", "Alta"]
)

df.head()

# 8. Remoção de duplicidades pelo identificador

df = df.drop_duplicates(subset=["id_venda"], keep="first").reset_index(drop=True)

# 9. Agrupamentos

print("\nFaturamento por categoria:")
print(df.groupby("categoria")["valor_total"].sum().sort_values(ascending=False))

print("\nFaturamento por cidade:")
print(df.groupby("cidade")["valor_total"].sum().sort_values(ascending=False))

print("\nQuantidade de vendas por faixa:")
print(df["faixa_venda"].value_counts())

# 10. Resultado final

print("\nDimensões após tratamento:", df.shape)
print("\nNulos após tratamento:")
print(df.isna().sum())
print("\nPrimeiras linhas:")
print(df.head())
