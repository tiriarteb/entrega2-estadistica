---
license: cc-by-4.0
task_categories:
  - tabular-classification
  - time-series-forecasting
tags:
  - prediction-markets
  - kalshi
  - finance
  - trading
  - event-contracts
size_categories:
  - 100M<n<1B
---

# Kalshi Prediction Market Trades & Markets

Complete public trade-level and market-level data from [Kalshi](https://kalshi.com), the first CFTC-regulated prediction market exchange in the United States.

## Dataset Summary

| | Count | Timespan |
|---|---|---|
| **Trades** | 154,505,005 | June 2021 – January 2026 |
| **Markets** | 17,464,713 | June 2021 – January 2026 |
| **Events** | 4,721,915 | |
| **Total contracts traded** | 34.6 billion | |

## Data Files

### `trades/`

16 parquet shards (~10M rows each), sorted by `created_time`.

| Column | Type | Description |
|---|---|---|
| `trade_id` | string | Unique trade identifier |
| `ticker` | string | Market ticker (joins to `markets.ticker`) |
| `count` | int64 | Number of contracts in the trade |
| `yes_price` | int64 | Price of YES contracts (1–99 cents) |
| `no_price` | int64 | Price of NO contracts (1–99 cents) |
| `taker_side` | string | Side the taker bought: `yes` or `no` |
| `created_time` | timestamp[us, UTC] | When the trade occurred |

### `markets/`

4 parquet shards (~5M rows each).

| Column | Type | Description |
|---|---|---|
| `ticker` | string | Unique market ticker |
| `event_ticker` | string | Parent event ticker (groups related markets) |
| `market_type` | string | Market type (e.g. `binary`) |
| `title` | string | Human-readable market title |
| `yes_sub_title` | string | Label for the YES side |
| `no_sub_title` | string | Label for the NO side |
| `status` | string | Market status: `settled`, `closed`, `active`, etc. |
| `yes_bid` | int64 | Best YES bid at time of fetch |
| `yes_ask` | int64 | Best YES ask at time of fetch |
| `no_bid` | int64 | Best NO bid at time of fetch |
| `no_ask` | int64 | Best NO ask at time of fetch |
| `last_price` | int64 | Last traded price |
| `volume` | int64 | Lifetime volume (contracts) |
| `volume_24h` | int64 | 24-hour volume |
| `open_interest` | int64 | Open interest |
| `result` | string | Resolution: `yes`, `no`, or empty if unresolved |
| `created_time` | timestamp | When the market was created |
| `open_time` | timestamp | When the market opened for trading |
| `close_time` | timestamp | When the market closed |

## Usage

```python
from datasets import load_dataset

ds = load_dataset("TrevorJS/kalshi-trades")

# or with DuckDB
import duckdb
con = duckdb.connect()
con.execute("SELECT * FROM 'hf://datasets/TrevorJS/kalshi-trades/trades/*.parquet' LIMIT 10")
```

## Data Quality

- **0 duplicate trades** (21,950 duplicates removed during cleaning)
- **0 null trade IDs or prices**
- **100% trade-to-market join coverage** (64 orphan tickers with no API-available market record were removed; 1,258 trades dropped, <0.001% of dataset)
- All timestamps are UTC

## Provenance

Trade data was collected from the [Kalshi public API](https://trading-api.readme.io/reference/getmarkettrades). Market metadata was backfilled using the `/markets` batch ticker endpoint. The dataset combines data originally collected by [Andrew Becker](https://huggingface.co/datasets/andrewbecker/kalshi_trades) (June 2021 – November 2025) with an incremental update through January 2026.

## Acknowledgments

- [Andrew Becker](https://huggingface.co/andrewbecker) for the original `kalshi_trades` dataset and collection infrastructure
- [Kalshi](https://kalshi.com) for providing a public API

## License

CC-BY-4.0
