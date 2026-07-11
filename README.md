# TPC-H Benchmark

Evaluate and compare database performance using the [TPC-H](https://www.tpc.org/tpch/) decision-support benchmark. This project generates TPC-H data with DuckDB, loads it into PostgreSQL, runs the official 22 TPC-H queries, and compares query execution times on Google Colaboratory.

## Overview

- Generate TPC-H datasets using DuckDB’s `dbgen` at a chosen scale factor (SF)
- Export all TPC-H tables to CSV and load them into PostgreSQL
- Run the official TPC-H query set (Q1–Q22)
- Benchmark and compare query latency between **DuckDB** and **PostgreSQL**

## Tech Stack

| Component | Tool |
|-----------|------|
| Data generation | DuckDB (`INSTALL tpch` / `CALL dbgen`) |
| Analytical DB | DuckDB |
| Relational DB | PostgreSQL |
| Environment | Google Colaboratory |
| Language | Python |
| Libraries | `duckdb`, `pandas`, `psycopg2-binary` |

## TPC-H Schema

The benchmark uses these tables:

- `region`, `nation`, `supplier`, `customer`
- `part`, `partsupp`, `orders`, `lineitem`

## How It Works

1. **Install dependencies** — DuckDB, pandas, PostgreSQL, and `psycopg2`
2. **Generate data** — `CALL dbgen(sf=5)` creates ~5 GB of TPC-H data in DuckDB
3. **Create PostgreSQL schema** — define all 8 TPC-H tables
4. **Export & load** — copy tables from DuckDB → CSV → PostgreSQL
5. **Load official queries** — fetch Q1–Q22 via DuckDB’s `tpch_queries()`
6. **Index & analyze** — create join indexes and run `ANALYZE`
7. **Benchmark** — time each query on DuckDB and PostgreSQL (5-minute timeout per PostgreSQL query)

## Setup (Google Colab)

1. Upload `TPC-H.ipynb` to Google Colab, or open it from Drive.
2. Run cells in order from the top.
3. Ensure PostgreSQL starts successfully before connecting:

```bash
service postgresql start
```

4. Default notebook password for the `postgres` user:

```text
tpchDB
```

> Change this password if you reuse the notebook outside Colab.

## Scale Factor

| Scale Factor (SF) | Approx. Data Size |
|-------------------|-------------------|
| 1 | ~1 GB |
| 5 | ~5 GB (default in notebook) |
| 10+ | Larger; may need more Colab memory/disk |

Change the scale factor here:

```python
duck_con.execute("CALL dbgen(sf=5)")
```

## Benchmark Output

For each query `Q1`–`Q22`, the notebook prints:

```text
Running Q1...
Q1 DuckDB: <seconds> PostgreSQL: <seconds | TIMEOUT>
```

PostgreSQL queries that exceed 5 minutes are marked as `TIMEOUT`.

## Project Files

```text
TPC-H.ipynb   # Main benchmark notebook
README.md     # Project documentation
```

Generated during execution (not committed):

```text
*.csv                 # Exported TPC-H tables
queries/              # Official SQL queries (if downloaded)
tpch-kit-master/      # Optional TPC-H kit download
```

## Notes

- Quote the `"orders"` table name in PostgreSQL because `orders` can conflict with reserved keywords.
- If `region` or `nation` row counts look wrong after load, use the truncate/reload cells in the notebook.
- For larger scale factors (e.g., SF 10–50), expect longer generation, load, and query times; Colab free tier may hit memory or disk limits.

## References

- [TPC-H Benchmark Specification](https://www.tpc.org/tpch/)
- [DuckDB TPC-H Extension](https://duckdb.org/docs/extensions/tpch)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
