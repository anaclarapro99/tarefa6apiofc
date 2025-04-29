import streamlit as st
import requests
import pandas as pd
import datetime

st.set_page_config(page_title="Cripto Dashboard", layout="wide")

# --- Título ---
st.title("📈 Painel de Criptomoedas em Tempo Real - CoinGecko API")

# --- Sidebar ---
st.sidebar.header("Configurações")
vs_currency = st.sidebar.selectbox("Moeda base", ["usd", "eur", "brl"])
per_page = st.sidebar.slider("Quantidade de moedas", min_value=5, max_value=100, value=10)
refresh = st.sidebar.button("🔄 Atualizar agora")

# --- Função para pegar dados da API ---
@st.cache_data(ttl=300)
def get_crypto_data(vs_currency, per_page):
    url = f"https://api.coingecko.com/api/v3/coins/markets"
    params = {
        "vs_currency": vs_currency,
        "order": "market_cap_desc",
        "per_page": per_page,
        "page": 1,
        "sparkline": "false"
    }
    response = requests.get(url, params=params)
    data = response.json()
    df = pd.DataFrame(data)[["id", "symbol", "current_price", "market_cap", "price_change_percentage_24h"]]
    df.rename(columns={
        "id": "Criptomoeda",
        "symbol": "Símbolo",
        "current_price": f"Preço ({vs_currency.upper()})",
        "market_cap": "Cap. de Mercado",
        "price_change_percentage_24h": "Variação 24h (%)"
    }, inplace=True)
    return df

# --- Obter dados ---
df_crypto = get_crypto_data(vs_currency, per_page)

# --- Exibir dados ---
st.dataframe(df_crypto, use_container_width=True)

# --- Gráfico ---
st.subheader("📊 Variação de Preço (24h)")
st.bar_chart(df_crypto.set_index("Criptomoeda")["Variação 24h (%)"])

# --- Atualização de tempo ---
st.caption(f"Última atualização: {datetime.datetime.now().strftime('%d/%m/%Y %H:%M:%S')}")
