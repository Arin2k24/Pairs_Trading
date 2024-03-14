# Pairs_Trading

## Overview

This Python script implements a pairs trading strategy using MACD (Moving Average Convergence Divergence) signals applied to z-score spreads between highly correlated cryptocurrency pairs. The strategy aims to profit from the temporary price divergences between two assets within a pair while maintaining a market-neutral position.

## Strategy

Pairs trading involves identifying two assets with a high degree of correlation and trading them based on their relative prices. The basic steps of the strategy are as follows:

1. **Data Collection**: Download historical price data for selected cryptocurrency pairs from Yahoo Finance or other sources.

2. **Correlation Analysis**: Calculate the correlation matrix between all pairs of cryptocurrencies. Identify highly correlated pairs with correlation coefficients above a specified threshold (e.g., 0.9).

3. **Spread Calculation**: Calculate the spread between the prices of each pair's constituent assets. Compute the z-score of the spread to normalize it.

4. **MACD Calculation**: Apply the MACD indicator to the z-score spread. Calculate the MACD line, signal line, and MACD histogram.

5. **Signal Generation**: Generate trading signals based on MACD crossovers. A bullish crossover occurs when the MACD line crosses above the signal line, indicating a potential long (buy) signal. Conversely, a bearish crossover occurs when the MACD line crosses below the signal line, signaling a potential short (sell) signal.

6. **Trade Execution**: Open long (buy) positions when a bullish crossover signal is detected and short (sell) positions when a bearish crossover signal is detected. Close positions when the opposite crossover signal is generated or according to predefined exit criteria.

## Example

Suppose we have two highly correlated cryptocurrency pairs: BTC-USD and ETH-USD. We calculate the z-score spread between the prices of BTC-USD and ETH-USD and apply the MACD indicator to the spread. When a bullish MACD crossover occurs (MACD line crosses above the signal line), we enter a long position on BTC-USD and a short position on ETH-USD. We exit the position when a bearish MACD crossover or other exit criteria are met.

## Usage

1. Clone this repository or download the Python script.
2. Install the required dependencies (`pandas`, `numpy`, `seaborn`, `matplotlib`, `yfinance`) using pip.
3. Run the Python script using a Python interpreter.

## Disclaimer

This trading strategy is for educational purposes only and should not be considered financial advice. Cryptocurrency trading involves risk, and past performance is not indicative of future results. Use this script at your own risk and discretion.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
