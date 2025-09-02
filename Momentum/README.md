# Momentum Trading Strategies

This repository contains implementations of momentum-based trading strategies that capitalize on short-term price movements triggered by specific market events.

## Overview

We implement two momentum trading strategies:
1. **Index Update Strategy** ✅ (Implemented)
2. **IPO Secondary Market Strategy** 🚧 (Future Implementation)

## 1. Index Update Strategy

### Key Idea

When stocks are added to or removed from major indices (like the S&P 500), they experience significant price momentum due to:
- **Index fund rebalancing**: Passive funds must buy added stocks and sell removed stocks
- **Market attention**: Increased visibility and analyst coverage
- **Supply/demand imbalance**: Institutional buying creates temporary momentum

Our analysis reveals that **1-day holding period is optimal**, capturing the immediate momentum effect with 871.8% returns over the backtest period.

### Data Preprocessing

#### Source Data
- S&P 500 index changes from 2015-2025
- Contains: Announcement Date, Effective Date, Added/Removed tickers, Reasons

#### Data Cleaning Pipeline
```python
# Filter for sufficient data period
df = df[df['Announcement_Date'] >= '2015-07-01']

# Handle missing tickers and corporate actions
df = df.dropna(subset=['Added_Ticker', 'Removed_Ticker'], how='all')

# Sort by announcement date for chronological backtesting
df = df.sort_values('Announcement_Date').reset_index(drop=True)
```

#### Price Data Integration
- Uses Yahoo Finance API for historical price data
- Implements NYSE trading calendar for accurate business day calculations
- Handles stock splits and dividends through adjusted prices

### Backtesting Methodology

#### Strategy 1: Long-Only Added Tickers
- **Capital**: $1,000,000 USD
- **Entry**: Buy stocks on announcement date (not effective date)
- **Position Sizing**: Full capital deployed per trade (compound returns)
- **Exit**: Sell after specified holding period
- **Risk Management**: Equal weight if multiple announcements

#### Strategy 2: Long/Short Pairs Trading
- **Capital**: $1,000,000 USD (50% long, 50% short)
- **Entry**: Only when both added and removed tickers exist
  - Long: Added ticker (captures upward momentum)
  - Short: Removed ticker (captures downward momentum)
- **Position Sizing**: Equal weight allocation
- **Exit**: Close both positions simultaneously

#### Holding Period Optimization
- **Range Tested**: 1-22 trading days
- **Calendar**: Uses NYSE trading calendar for precise business day counting
- **Performance Metrics**: 
  - Total Return
  - Sharpe Ratio
  - Maximum Drawdown
  - Win Rate

### Key Results

#### Optimal Holding Periods
| Strategy | Best Return | Best Sharpe | Optimal Period |
|----------|-------------|-------------|----------------|
| Strategy 1 (Long Only) | 871.8% | 4.62 | **1 day** |
| Strategy 2 (Long/Short) | 117.8% | 2.76 | 16 days |

#### Strategy 1 Performance (1-day holding)
- **Total Return**: 871.8%
- **Sharpe Ratio**: 4.62
- **Successful Trades**: 170/223 announcements
- **Win Rate**: 50.5%
- **Max Drawdown**: -75.84%

#### Key Insight
**1-day holding period is optimal** for capturing index addition momentum, supporting the hypothesis that the effect is immediate and strong.

### Implementation Files

```
IndexUpdate/
├── StrategyDesign_Daily.ipynb          # Sequential implementation
├── StrategyDesign_Multithread.ipynb    # Parallel processing version
├── data/
│   └── sp500_index_changes.csv        # Historical S&P 500 changes
└── log/                               # Execution logs and results
    ├── trading_log_hp*.log            # Trade-by-trade logs
    └── holding_period_analysis.csv    # Optimization results
```

### Usage

#### Quick Start
```python
# Load and run basic analysis
python -c "
import pandas as pd
df = pd.read_csv('./data/sp500_index_changes.csv')
# Run StrategyDesign_Daily.ipynb for full analysis
"
```

#### Notebooks
- **Daily Notebook**: Sequential processing, detailed logging
- **Multithread Notebook**: Parallel processing for faster execution

### Results Visualization

The analysis produces comprehensive visualizations:
- **Performance Evolution**: Portfolio value over time
- **Holding Period Optimization**: Returns vs. holding period curves
- **Risk-Return Profiles**: Scatter plots of risk vs. return
- **Distribution Analysis**: Histograms of trade returns
- **Heatmaps**: Performance across different holding periods

## 2. IPO Secondary Market Strategy (Future Implementation)

### Planned Approach
- **Data Source**: IPO announcements and secondary market listings
- **Momentum Hypothesis**: Post-IPO momentum from institutional interest
- **Implementation**: Similar backtesting framework as Index Update strategy

## Technical Architecture

### Data Pipeline
```
Raw Data → Preprocessing → Price Fetching → Strategy Execution → Results Analysis
```

### Key Functions
- `get_trading_end_date()`: NYSE calendar integration
- `get_stock_prices()`: Yahoo Finance data fetching with error handling
- `calculate_strategy_returns_with_portfolio()`: Core backtesting engine
- `calculate_sharpe_ratio()`: Risk-adjusted performance metrics

## Future Improvements

### 1. Performance Optimization (Multithreading)
**Current Issue**: Sequential processing takes ~180 seconds
**Planned Solution**: 
```python
# Parallel price fetching
with ThreadPoolExecutor(max_workers=8) as executor:
    futures = {executor.submit(get_stock_prices, ticker, start, end): ticker 
               for ticker in unique_tickers}
```

**Expected Improvement**: 60-80% speed reduction

### 2. Advanced Risk Management (Stop Loss)
**Current Limitation**: Buy-and-hold for entire holding period
**Planned Enhancement**:
- Intraday stop-loss orders (requires higher frequency data)
- Trailing stops to capture momentum while limiting downside
- Dynamic position sizing based on volatility

**Data Requirements**: 
- Minute-level price data
- Real-time market data feeds
- Enhanced execution modeling

### 3. Additional Features
- **Transaction Costs**: Realistic spread and commission modeling
- **Market Impact**: Price impact of large orders
- **Regime Detection**: Bull/bear market adaptation
- **Multi-Asset**: Extend to other indices (Russell 2000, NASDAQ)

## Dependencies

```python
pandas>=1.3.0
numpy>=1.20.0
yfinance>=0.1.70
matplotlib>=3.3.0
pandas-market-calendars>=2.1.0
concurrent.futures  # Built-in Python
```

## Performance Benchmarks

| Metric | Current Implementation | Target (Optimized) |
|--------|----------------------|-------------------|
| Execution Time | 180 seconds | <60 seconds |
| Memory Usage | ~500MB | <300MB |
| CPU Utilization | Single core | Multi-core |

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/improvement`)
3. Implement changes with proper testing
4. Submit pull request with performance benchmarks

## License

This project is for educational and research purposes. No investment advice is provided.

---

**⚠️ Disclaimer**: Past performance does not guarantee future results. This is a research implementation and should not be used for actual trading without proper risk management and compliance review.