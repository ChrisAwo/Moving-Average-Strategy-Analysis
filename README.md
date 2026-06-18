
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

2. Calculate moving averages and trading signals, and track performance metrics such as returns, drawdowns, holding periods, and win rates.
```python
        # -------- BASIC DATA --------
        price['Asset'] = ticker
        price['Returns'] = price['Close'] / price['Close'].shift(1)
        price.loc[0, 'Returns'] = 1
        price['Bench_Bal'] = starting_balance * price['Returns'].cumprod()

        # -------- STRATEGY --------
        price['MA_Series'] = price['Close'].rolling(window=ma).mean()
        price['Longs'] = price['Close'] > price['MA_Series']

        price['Sys_Ret'] = np.where(price['Longs'].shift(1) == True, price['Returns'], 1)
        price['Sys_Bal'] = starting_balance * pd.Series(price['Sys_Ret']).cumprod()

        price['MA_Type'] = ma
        price['Bench_Peak'] = price.Bench_Bal.cummax()
        price['Bench_DD'] = price.Bench_Bal - price.Bench_Peak
        price['max_dd'] = ((price.Bench_DD / price.Bench_Peak).min())
```

3. Simulate trades based on entry and exit rules.
```python
        # -------- ENTRY / EXIT --------
        price['Entry'] = (price['Longs'] == True) & (price['Longs'].shift(1) == False)
        price['Exit']  = (price['Longs'] == False) & (price['Longs'].shift(1) == True)

        # -------- EXTRACT TRADES --------
        entries = price.loc[price['Entry']].copy()
        exits   = price.loc[price['Exit']].copy()

        entries = entries.reset_index(drop=True)
        exits   = exits.reset_index(drop=True)

        min_len = min(len(entries), len(exits))
        entries = entries.iloc[:min_len]
        exits   = exits.iloc[:min_len]
```

4. Store and organize results.
   
```python
all_prices.append(price)
all_trades.append(trades)

final_data = pd.concat(all_prices, ignore_index=True)
trades_data = pd.concat(all_trades, ignore_index=True)
```

5. Connect SQL data to Power BI for visualization and analysis.
   
```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://root:root@localhost/ma_analysis")

final_data.to_csv('price_data.csv', index=False)
trades.to_csv('trades_data.csv', index=False)

final_data.to_sql(name='price_data', con=engine, if_exists='replace', index=False)
trades_data.to_sql(name='trades_data', con=engine, if_exists='replace', index=False)
```

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

Blocks of code below:

```sql
CREATE TABLE executive_summary AS
WITH trade_stats AS (
    SELECT
        MA_Type,
        COUNT(*) AS Total_Trades,
        round(AVG(Trade_Return), 4) AS Avg_Return,
        round(SUM(Trade_Return), 4) AS Total_Return,
        round(AVG(CASE WHEN Result = 'Win' THEN 1 ELSE 0 END) * 100, 2) AS Win_Rate,
        round(max(Holding_Period), 2) as Max_Holding,
		round(avg(Holding_Period), 2) as Avg_Holding
    FROM trades_data
    GROUP BY MA_Type
),

drawdown_stats AS (
    SELECT
        MA_Type,
        MIN((Sys_Bal - Bench_Bal) / Bench_Bal) AS Max_Drawdown,
        (Sys_Bal - Bench_Peak) / Bench_Peak AS Drawdown
    FROM price_data
    GROUP BY MA_Type, Drawdown
)

SELECT 
    t.*,
    d.Max_Drawdown
FROM trade_stats t
JOIN drawdown_stats d
ON t.MA_Type = d.MA_Type;

select * from executive_summary;


CREATE TABLE return_distribution AS
WITH stats AS (
    SELECT
        MA_Type,
        Trade_Return,

        AVG(Trade_Return) OVER (
            PARTITION BY MA_Type
        ) AS Mean_Return,

        STDDEV(Trade_Return) OVER (
            PARTITION BY MA_Type
        ) AS Std_Return
    FROM trades_data
)

SELECT
    MA_Type,
    Trade_Return,

    CASE 
        WHEN Std_Return = 0 THEN NULL
        ELSE (Trade_Return - Mean_Return) / Std_Return
    END AS Z_Score,

    RANK() OVER (
        PARTITION BY MA_Type 
        ORDER BY Trade_Return DESC
    ) AS Return_Rank
FROM stats;

select * from return_distribution;

```
---

## Key Takeaway

Although this was an early project, it provided valuable insights into strategy evaluation and performance analysis. It also helped me develop skills in Python, SQL, and Power BI while building the foundation for more advanced quantitative trading projects.
