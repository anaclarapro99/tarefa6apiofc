import streamlit as st
import requests
import pandas as pd

st.set_page_config(page_title="Cripto Dashboard", layout="wide")
st.title("📊 Dashboard de Criptomoedas (via CoinGecko API)")

# Sidebar
st.sidebar.header("Configurações")
moeda = st.sidebar.selectbox("Moeda", ["usd", "eur", "brl"])
quantidade = st.sidebar.slider("Número de moedas", 5, 50, 10)

# Função para obter dados da API CoinGecko
def carregar_dados(moeda, quantidade):
    url = "https://api.coingecko.com/api/v3/coins/markets"
    parametros = {
        "vs_currency": moeda,
        "order": "market_cap_desc",
        "per_page": quantidade,
        "page": 1,
        "sparkline": False
    }
    resposta = requests.get(url, params=parametros)
    if resposta.status_code == 200:
        dados = resposta.json()
        df = pd.DataFrame(dados)
        df = df[["name", "symbol", "current_price", "market_cap", "price_change_percentage_24h"]]
        df.columns = ["Nome", "Símbolo", f"Preço ({moeda.upper()})", "Capitalização de Mercado", "Variação 24h (%)"]
        return df
    else:
        st.error("Erro ao buscar dados da API.")
        return pd.DataFrame()

# Carregar dados
df = carregar_dados(moeda, quantidade)

# Mostrar tabela
if not df.empty:
    st.dataframe(df, use_container_width=True)
    st.bar_chart(df.set_index("Nome")["Variação 24h (%)"])
