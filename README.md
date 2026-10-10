# TPC-H Benchmark

Latency comparison of DuckDB and PostgreSQL on the official [TPC-H](https://www.tpc.org/tpch/) analytical query set. The notebook builds the data in DuckDB, loads it into PostgreSQL, and times all 22 queries on the same machine. It is written to run on Google Colab.

## What it does

1. Install DuckDB, pandas, PostgreSQL, and `psycopg2`
2. Generate data with `CALL dbgen(sf=5)` (about 5 GB)
3. Create the eight TPC-H tables in PostgreSQL
4. Export DuckDB tables to CSV and load them into PostgreSQL
5. Load Q1–Q22 from DuckDB `tpch_queries()`
6. Add join indexes and run `ANALYZE`
7. Time each query on both engines. PostgreSQL is stopped at 5 minutes and marked `TIMEOUT`

Tables: `region`, `nation`, `supplier`, `customer`, `part`, `partsupp`, `orders`, `lineitem`.

| Piece | Tool |
| --- | --- |
| Data generation and analytical engine | DuckDB (`INSTALL tpch`, `CALL dbgen`) |
| Relational engine | PostgreSQL |
| Notebook runtime | Google Colab |
| Libraries | `duckdb`, `pandas`, `psycopg2-binary` |

## Setup

1. Open `TPC-H.ipynb` in Colab, or upload it.
2. Run the cells from the top.
3. Start PostgreSQL before the connection cell:

```bash
service postgresql start
```

The notebook’s `postgres` password is `tpchDB`. Change it if you run this outside Colab.

Default scale factor:

```python
duck_con.execute("CALL dbgen(sf=5)")
```

| Scale factor | Approx. size |
| --- | --- |
| 1 | ~1 GB |
| 5 | ~5 GB (notebook default) |
| 10+ | Larger; Colab may run out of disk or memory |

## Output

```text
Running Q1...
Q1 DuckDB: <seconds> PostgreSQL: <seconds | TIMEOUT>
```

## Notes

- Quote `"orders"` in PostgreSQL. Unquoted `orders` can hit a reserved word. Quote only a bare `orders` word, so a second pass does not turn `"orders"` into `""orders""`.
- If `region` or `nation` row counts look wrong after load, use the truncate and reload cells.
- CSV exports, a `queries/` folder, and `tpch-kit-master/` are produced while the notebook runs. They are not part of the repo.
- The timed comparison is the same 22 queries on DuckDB and PostgreSQL at scale factor 5.

## Files

```text
TPC-H.ipynb                # the benchmark notebook
TPC.docx                   # written report of the results
TPC_H_Interview_Guide.html # walkthrough of the design and the numbers
```

## References

- [TPC-H specification](https://www.tpc.org/tpch/)
- [DuckDB TPC-H extension](https://duckdb.org/docs/extensions/tpch)
- [PostgreSQL documentation](https://www.postgresql.org/docs/)
