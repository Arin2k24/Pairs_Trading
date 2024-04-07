# Pairs Trading Strategy

Pairs trading is a market-neutral trading strategy that aims to profit from the relative performance of two correlated assets. In this strategy, we identify pairs of stocks that exhibit high correlation and trade based on deviations from their historical relationship.

## Strategy Overview

### Objective

The objective of this pairs trading strategy is to identify highly correlated pairs of stocks, calculate trading signals based on statistical indicators, manage positions, and implement risk control measures.

### Strategy Components

1. **Data Collection**: Historical stock price data for selected stocks is collected using the Yahoo Finance API.

2. **Pair Selection**: Pairs of stocks with high correlation coefficients are identified.

3. **Spread Calculation**: The spread between the two stocks in each pair is calculated using a defined hedge ratio obtained from linear regression.

4. **Stationarity Test**: The Dickey-Fuller test is performed on the spread to determine stationarity, indicating a mean-reverting behavior suitable for pairs trading.

5. **Z-Score Calculation**: The z-score of the spread is calculated using a rolling window for mean and standard deviation.

6. **Entry and Exit Signals**: Trading signals are generated based on z-score deviations from historical means. Long and short positions are initiated when the z-score crosses predefined thresholds.

7. **MACD Indicator**: An additional trading signal is generated using the MACD indicator applied to the z-score spread.

8. **Position Management**: Positions are managed dynamically, and stop-loss measures are implemented to control risk.

9. **Performance Monitoring**: Profit and loss (PnL) are monitored over time to evaluate the effectiveness of the strategy.

## Pros and Cons

### Pros

- Market-Neutral: Pairs trading is designed to be market-neutral, meaning it can potentially generate profits regardless of market direction.
- Statistical Edge: The strategy relies on statistical analysis to identify trading opportunities based on mean-reversion principles, offering a systematic approach to trading.
- Risk Control: Stop-loss measures help control risk by limiting losses in case of adverse price movements.
- Diversification: By trading multiple pairs of stocks, the strategy can provide diversification benefits and reduce idiosyncratic risk.

### Cons

- Correlation Breakdown: Pairs trading relies on the historical correlation between assets, which may break down during periods of market stress or fundamental changes in the underlying stocks.
- Execution Challenges: Trading pairs requires simultaneous execution of buy and sell orders, which can be challenging, especially for illiquid stocks or during volatile market conditions.
- Model Risk: The strategy's performance heavily depends on the accuracy of statistical models used for spread calculation and signal generation, introducing model risk.
- Transaction Costs: Frequent trading in multiple stocks incurs transaction costs, which can erode profits, especially for smaller portfolios.

## References

- [Pairs Trading Basics](https://blog.quantinsti.com/pairs-trading-basics/)
- [MACD Indicator](https://www.investopedia.com/terms/m/macd.asp)

## Requirements

- Python 3.x
- Required libraries: `yfinance`, `pandas`, `numpy`, `seaborn`, `matplotlib`, `statsmodels`
- Access to financial data via Yahoo Finance API

## Usage

1. Install the required Python libraries.
2. Run the provided Python script to execute the pairs trading strategy.
3. Adjust parameters such as window sizes, threshold values, and stop-loss levels as needed.
4. Monitor the PnL and adjust the strategy accordingly.

## Code
```python
# Calculate correlation matrix
correlation_matrix = stock_data.corr()

# Plot correlation matrix as a heatmap
plt.figure(figsize=(12, 8))
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', fmt=".2f", linewidths=0.5)
plt.title('Stocks Correlation Matrix')
plt.show()
```
![image](https://github.com/Arin2k24/Pairs_Trading/assets/157686042/42cbbf3c-404a-4c74-9878-6c84184ffa57)
```python
# Initialize an empty dictionary to store pairs
high_correlation_pairs = {}

# Iterate through the correlation matrix to find pairs with correlation > 0.9 (or < -0.9) and not the same coin
for i in range(len(correlation_matrix.columns)):
    for j in range(i+1, len(correlation_matrix.columns)):
        if (correlation_matrix.iloc[i, j] > 0.95) and \
                correlation_matrix.columns[i] != correlation_matrix.columns[j]:
            pair = (correlation_matrix.columns[i], correlation_matrix.columns[j])
            high_correlation_pairs[pair] = correlation_matrix.iloc[i, j]

# Display the dictionary of pairs
print(high_correlation_pairs)
```
```python
# Function to calculate spread
def calculate_spread(a, b, hedge_ratio):
    spread = np.log(a) - hedge_ratio * np.log(b)
    return spread

# Dictionary to store hedge ratios for each pair
hedge_ratios = {}

# Calculate hedge ratios using regression for each pair
for pair in high_correlation_pairs:
    # Preprocess data to handle missing or infinite values
    df = stock_data[[pair[0], pair[1]]].dropna()
    x = np.log(df[pair[0]])
    y = np.log(df[pair[1]])

    # Perform OLS regression
    X = sm.add_constant(x)
    model = sm.OLS(y, X).fit()

    # Store hedge ratio if regression is successful
    if not model.params.empty:
        hedge_ratios[pair] = model.params[1]
    else:
        print(f"Regression failed for pair: {pair}")

# Print hedge ratios for each pair
print("Hedge Ratios:")
for pair, ratio in hedge_ratios.items():
    print(f"{pair}: {ratio}")
```
```python
# Calculate and print spreads for each pair using the calculated hedge ratio
print("\nSpreads:")
for pair, ratio in hedge_ratios.items():
    spread = calculate_spread(stock_data[pair[0]], stock_data[pair[1]], ratio)
    print(f"{pair}: Spread mean: {spread.mean()}, Spread std dev: {spread.std()}")
```
```python
# Dictionary to store acceptable pairs
acceptable_pairs = {}

# Run Dickey Fuller test on the spread values
for pair, ratio in hedge_ratios.items():
    # Preprocess data to handle missing or infinite values
    df = pd.concat([stock_data[pair[0]], stock_data[pair[1]]], axis=1).dropna()
    spread = calculate_spread(df[pair[0]], df[pair[1]], ratio)
    result = adfuller(spread)

    # Check if p-value is less than 0.05
    if result[1] < 0.05:
        acceptable_pairs[pair] = hedge_ratios[pair]

    print(f"\nDickey-Fuller Test Results for {pair}:")
    print('ADF Statistic:', result[0])
    print('p-value:', result[1])
    print('Critical Values:')
    for key, value in result[4].items():
        print(f'   {key}: {value}')
```
```python
# Function to calculate z-score
def calculate_zscore(spread, window):
    rolling_mean = spread.rolling(window=window).mean()
    rolling_std = spread.rolling(window=window).std()
    zscore = (spread - rolling_mean) / rolling_std
    return zscore
```
```python
# Entry and exit points
window = 20  # Window size for rolling mean and std deviation
threshold_upper = 2  # Upper threshold for z-score entry
threshold_lower = -2  # Lower threshold for z-score entry

# Entry and exit points for each pair
entry_long_points_dict = {}
entry_short_points_dict = {}
exit_points_dict = {}

for pair, ratio in acceptable_pairs.items():
    spread = calculate_spread(stock_data[pair[0]], stock_data[pair[1]], ratio)
    zscore = calculate_zscore(spread, window)
    open_trade = False

    # Initialize lists to store entry and exit points for this pair
    entry_long_points = []
    entry_short_points = []
    exit_points = []

    # Find entry and exit points
    for i in range(len(zscore)):
        if not open_trade:
            if zscore[i] <= threshold_lower:  # Entry Long
                entry_long_points.append((stock_data['index'][i], zscore[i]))
                open_trade = True
            elif zscore[i] >= threshold_upper:  # Entry Short
                entry_short_points.append((stock_data['index'][i], zscore[i]))
                open_trade = True
        elif open_trade:
            if (zscore[i] > 0 and zscore[i-1] <= 0) or (zscore[i] < 0 and zscore[i-1] >= 0):  # Exit Long or Short
                exit_points.append((stock_data['index'][i], zscore[i]))
                open_trade = False

    # Store entry and exit points for this pair in the dictionaries
    entry_long_points_dict[pair] = entry_long_points
    entry_short_points_dict[pair] = entry_short_points
    exit_points_dict[pair] = exit_points
```
```python
# Plot spread, z-score, and MACD for each pair with entry and exit points
for pair, ratio in acceptable_pairs.items():
    spread = calculate_spread(stock_data[pair[0]], stock_data[pair[1]], ratio)
    zscore = calculate_zscore(spread, window)

    # Calculate MACD
    short_ema = zscore.ewm(span=12, adjust=False).mean()
    long_ema = zscore.ewm(span=26, adjust=False).mean()
    macd_line = short_ema - long_ema
    signal_line = macd_line.ewm(span=9, adjust=False).mean()
    macd_histogram = macd_line - signal_line

    # Plot spread, z-score, and MACD
    plt.figure(figsize=(12, 6))
    plt.plot(stock_data['index'], spread, label='Spread')
    plt.axhline(threshold_upper, color='r', linestyle='--', label='Upper Threshold')
    plt.axhline(threshold_lower, color='g', linestyle='--', label='Lower Threshold')
    plt.plot(stock_data['index'], zscore, label='Z-Score', color='orange')
    plt.title(f"Spread, Z-Score, and MACD for {pair[0]} and {pair[1]}")
    plt.xlabel('Date')
    plt.ylabel('Value')
    plt.legend()

    # Plot entry and exit points
    entry_long_points = entry_long_points_dict.get(pair, [])
    entry_short_points = entry_short_points_dict.get(pair, [])
    exit_points = exit_points_dict.get(pair, [])

    for point in entry_long_points:
        plt.scatter(point[0], point[1], color='green', marker='^')
    for point in entry_short_points:
        plt.scatter(point[0], point[1], color='red', marker='v')
    for point in exit_points:
        plt.scatter(point[0], point[1], color='blue', marker='o')

    # Add legend
    legend_entries = {'Entry Long': 'green', 'Entry Short': 'red', 'Exit Trade': 'blue'}
    plt.legend(labels=legend_entries.keys(), handles=[plt.Line2D([0], [0], marker='^', color='w', markerfacecolor=color, markersize=10) for color in legend_entries.values()], loc='best')

    # Plot MACD
    plt.figure(figsize=(12, 6))
    plt.plot(stock_data['index'], macd_line, label='MACD Line', color='blue')
    plt.plot(stock_data['index'], signal_line, label='Signal Line', color='red')
    plt.bar(stock_data['index'], macd_histogram, label='MACD Histogram', color='green')
    plt.title('MACD Indicator for Z-Score Spread')
    plt.xlabel('Date')
    plt.ylabel('Value')
    plt.legend()
    plt.show()
```
![image](https://github.com/Arin2k24/Pairs_Trading/assets/157686042/ff915096-5126-466d-b7ce-2c6023f8058f)
![image](https://github.com/Arin2k24/Pairs_Trading/assets/157686042/d3da17f3-f8d7-4ae8-86b8-0c755b96af7f)
![image](https://github.com/Arin2k24/Pairs_Trading/assets/157686042/44b85f43-b8d3-43da-93b7-3abed4f1bff1)

## Disclaimer

- This pairs trading strategy is for educational purposes only and does not constitute financial advice.
- Past performance is not indicative of future results. Trading involves risks, and losses may occur.
- Use this strategy at your own risk, and consider consulting a financial advisor before making trading decisions.
