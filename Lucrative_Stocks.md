```python
# Read the data from AAPL, AMZN, and GOOG

aapl_data = pd.read_csv("aapl_historical.csv", index_col="Date", parse_dates=True)
amzn_data = pd.read_csv("amzn_historical.csv", index_col="Date", parse_dates=True)
goog_data = pd.read_csv("goog_historical.csv", index_col="Date", parse_dates=True)

# Extract closing prices

aapl_close = aapl_data['Close']
amzn_close = amzn_data['Close']
goog_close = goog_data['Close']

# Create a new DataFrame with AAPL, AMZN, and GOOG closing prices

df_lucrative_stocks = pd.DataFrame({
    "AAPL": aapl_close,
    "AMZN": amzn_close,
    "GOOG": goog_close
})

# Drop any NaNs in the DataFrame

df_lucrative_stocks.dropna(inplace=True)

# Calculate daily returns

df_lucrative_stocks_returns = df_lucrative_stocks.pct_change()

# Drop any resulting NaNs

df_lucrative_stocks_returns.dropna(inplace=True)

# Set weights for the stocks, assume we invested 40% in AAPL, 30% in AMZN and 30% in GOOG

weights = [0.4, 0.3, 0.3]

# Calculate weighted returns

df_lucrative_stocks_weighted_returns = df_lucrative_stocks_returns.dot(weights)

# Convert the Series to a DataFrame

df_lucrative_stocks_weighted_returns = df_lucrative_stocks_weighted_returns.to_frame()

# Rename the column 

df_lucrative_stocks_weighted_returns.columns = ['Lucrative_Stocks']

# Join the lucrative stocks portfolio returns to the DataFrame with Whale, Algo and S&P 500 returns

df_combined = pd.concat([df_whale_returns, df_algo_returns, df_sp500_returns, df_lucrative_stocks_weighted_returns], axis='columns', join='inner')


```
