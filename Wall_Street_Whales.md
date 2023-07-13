```python
# Initial imports
import pandas as pd
import numpy as np
import datetime as dt
from pathlib import Path
import matplotlib.pyplot as plt
%matplotlib inline

# Helper function to help clean data
def load_and_clean_data(file_path):
    # Reading the data
    df = pd.read_csv(file_path, parse_dates=True, index_col='Date', infer_datetime_format=True).sort_index()
    
    # Count the number of nulls in the DataFrame
    null_counts = df.isnull().sum()
    print(null_counts)
    
    # Drop rows containing nulls from the DataFrame
    df = df.dropna()
    
    # Verify that nulls have been dropped
    null_counts = df.isnull().sum()
    print(null_counts)
    return df

# Data Loading and Cleaning
try:
    whale_returns = load_and_clean_data('whale_returns.csv')
    algo_returns = load_and_clean_data('algo_returns.csv')
    sp500_history = load_and_clean_data('sp500_history.csv')
except FileNotFoundError as fnf_error:
    print(fnf_error)

# Rename `Close` Column to be specific to this portfolio.
sp500_history.rename(columns={"Close": "S&P 500"}, inplace=True)

# Calculate Daily Returns for S&P 500
sp500_history = sp500_history.pct_change().dropna()

# Combine Whale, Algorithmic, and S&P 500 Returns
combined_df = pd.concat([whale_returns, algo_returns, sp500_history], axis="columns", join="inner")

# Performance Analysis
# Daily Returns
daily_returns = combined_df.pct_change()
daily_returns.plot(title='Daily Returns')

# Cumulative Returns
cumulative_returns = (1 + daily_returns).cumprod()
cumulative_returns.plot(title='Cumulative Returns')

# Risk Analysis
# Standard Deviation
std_dev = daily_returns.std()
print(std_dev)

# Create a box plot for each portfolio
daily_returns.boxplot(grid=False, figsize=(20,10))

# Determine which portfolios are riskier than the S&P 500
riskier_than_sp500 = std_dev[std_dev > std_dev['S&P 500']]
print(riskier_than_sp500)

# Annualized Standard Deviation
annualized_std_dev = std_dev * np.sqrt(252)
print(annualized_std_dev)

# Rolling Statistics
rolling_std_dev = daily_returns.rolling(window=21).std()
rolling_std_dev.plot(title='21 Day Rolling Standard Deviation')

# Calculate the correlation
correlation = daily_returns.corr()
print(correlation)

# Choose one portfolio, for example, "Algo 1"
covariance = daily_returns['Algo 1'].cov(daily_returns['S&P 500'])
variance = daily_returns['S&P 500'].var()
beta = covariance / variance
print(f'Beta of Algo 1: {beta}')

# Exponentially Weighted Moving Average
ewm = daily_returns.ewm(span=21).mean()
ewm.plot(title='Exponentially Weighted Moving Average')

# Sharpe Ratios
sharpe_ratios = (daily_returns.mean() * 252) / (daily_returns.std() * np.sqrt(252))
sharpe_ratios.plot(kind='bar', title='Sharpe Ratios')

# Custom Portfolio
# Assuming you chose 'AAPL', 'COST', 'GOOG' as your stocks
try:
    aapl_data = load_and_clean_data('aapl_historical.csv')
    cost_data = load_and_clean_data('cost_historical.csv')
    goog_data = load_and_clean_data('goog_historical.csv')
except FileNotFoundError as fnf_error:
    print(fnf_error)

custom_portfolio = pd.concat([aapl_data['Close'], cost_data['Close'], goog_data['Close']], axis="columns", join="inner")
custom_portfolio.columns = ['AAPL', 'COST', 'GOOG']

# Calculate weighted returns
weights = np.array([0.4, 0.4, 0.2])  # Adjust weights according to your preference
custom_daily_returns = custom_portfolio.pct_change().dropna().dot(weights)

# Combine custom portfolio returns with original DataFrame
combined_df = pd.concat([combined_df, custom_daily_returns], axis="columns", join="inner")
combined_df.rename(columns={0: 'Custom Portfolio'}, inplace=True)
'''
