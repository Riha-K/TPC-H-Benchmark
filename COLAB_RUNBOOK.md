# TPC-H Colab Runbook

Practical checklist for running the DuckDB vs PostgreSQL notebook on Google Colab (SF=5). Use this when you need to re-run results or explain the workflow in an interview without scrolling the whole notebook.

## Before you start

- Open `TPC-H.ipynb` in Colab (or Jupyter with enough disk/RAM).
- Expect several GB for SF=5 generation + CSV export + Postgres load.
- Keep the 5-minute per-query timeout in mind for PostgreSQL on heavy queries.

## End-to-end steps

1. **Install** DuckDB, pandas, PostgreSQL client libraries, and `psycopg2` (as in the notebook).
2. **Generate** TPC-H data in DuckDB with `CALL dbgen(sf=5)`.
3. **Create** the eight tables in PostgreSQL (`region`, `nation`, `supplier`, `customer`, `part`, `partsupp`, `orders`, `lineitem`).
4. **Export / load** DuckDB tables → CSV → PostgreSQL (the notebook’s bridge path).
5. **Load queries** Q1–Q22 from DuckDB `tpch_queries()` (watch dialect / quoting quirks).
6. **Index + ANALYZE** join keys on PostgreSQL before timing.
7. **Time** each query on both engines; mark PostgreSQL `TIMEOUT` after 5 minutes.

## How to talk about results

- DuckDB is a **columnar OLAP** engine — strong on analytical scans/joins at SF=5 on one machine.
- PostgreSQL is a **row-store** engine — fair comparisons need tuning (`work_mem`, indexes, warm cache).
- Do **not** claim “Postgres is always slower.” Say: under *this* setup and timeout, columnar wins on the heavy TPC-H queries.

## Common failure modes

| Symptom | Likely cause | What to try |
| --- | --- | --- |
| Out of disk / RAM on Colab | SF=5 + CSV copies | Restart runtime; free `/tmp`; avoid duplicate exports |
| Query text errors on PG | Dialect / quoting (e.g. Q13) | Use the notebook’s rewritten SQL for Postgres |
| All PG queries “slow” | Cold cache, tiny `work_mem` | Warm-up run; raise `work_mem`; confirm indexes exist |
| One query hangs | Heavy join without good plan | `EXPLAIN`; add covering indexes; keep the timeout |

## Fairness disclaimer (say this)

Default Colab Postgres settings and the CSV load path bias the comparison toward DuckDB. Next experiments: warm vs cold cache, median of several runs, SF=1/5/10 curves, and tuned Postgres.

## Related files

- `TPC-H.ipynb` — executable benchmark
- `README.md` — short project overview
- `TPC_H_Interview_Guide.html` — interview pitch and Q&A
