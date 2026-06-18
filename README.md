
# Moving Average Strategy Analysis


## Project Overview

This project explores how different moving average strategies perform across a selection of stocks, including Apple, Tesla, and Microsoft.

The goal was to compare the profitability, risk, and trading behavior of different moving average periods (MA50, MA100, and MA200) and identify which strategy delivered the best overall performance.

This was one of my first quantitative finance projects and served as the foundation for my later work on more advanced backtesting systems.

---

## Tools Used

* Python (Pandas, NumPy)
* SQL
* Power BI
* SQLAlchemy
* Yahoo Finance API

---

## Project Workflow

The project followed the following process:

1. Download historical stock data using Python.

```python
for ma in ma_list:
    for ticker in tickers:
        
        price = yf.download(ticker, start=start, end=end, interval=interval)

        price.columns = price.columns.get_level_values(0)
        
        price = price.reset_index()
```

   
3. Calculate moving averages and trading signals.
4. Simulate trades based on entry and exit rules.
5. Track performance metrics such as returns, drawdowns, holding periods, and win rates.
6. Store and organize results using SQL.
7. Connect SQL data to Power BI for visualization and analysis.

---

## Dashboard Overview

The dashboard compares multiple moving average strategies and highlights:

* Final account balance
* Strategy profitability
* Maximum drawdown
* Number of trades
* Average holding period
* Win rate
* Asset performance

---

## What I Found

### Which strategy made the most money?

The MA50 strategy generated the highest overall profit. It reached the highest peak equity and finished with the strongest final account balance, making it the most profitable strategy in this analysis.

### Which strategy had the highest risk?

The MA100 strategy experienced the largest drawdown during the testing period, meaning it lost the greatest percentage of value during market declines.

### Which strategy offered the best balance between return and risk?

The MA50 strategy provided the strongest balance between profitability and risk. It delivered the highest returns while maintaining a lower drawdown than some of the other strategies.

### Which strategy was the most active?

The MA50 strategy executed the highest number of trades and had the shortest average holding period, showing a more active trading style.

### Which strategy required the most patience?

The MA200 strategy had the longest average holding period. Trades stayed open for much longer, making it the most patient and slower-moving strategy.

---

## SQL Analysis

SQL was used to organize and analyze the trading results before connecting them to Power BI.

Some of the SQL techniques used in this project include:

* Common Table Expressions (CTEs)
* Aggregate Functions (SUM, AVG, MAX)
* CASE Statements
* GROUP BY
* Window Functions
* Ranking Functions

Example:

```sql
WITH TradeStats AS (
    SELECT
        MA_Type,
        AVG(Trade_Return) AS Avg_Return
    FROM Trades
    GROUP BY MA_Type
)
SELECT *
FROM TradeStats;
```

---

## Key Takeaway

Although this was an early project, it provided valuable insights into strategy evaluation and performance analysis. It also helped me develop skills in Python, SQL, and Power BI while building the foundation for more advanced quantitative trading projects.
