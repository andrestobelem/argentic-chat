# DuckDB — Base de Datos Analítica In-Process

OLAP embebido sin servidor. "El SQLite del análisis de datos". Almacenamiento columnar, ejecución vectorizada, multi-threading automático.

## Instalación

```bash
uv add duckdb
```

## API básica

```python
import duckdb

# In-memory
con = duckdb.connect()
con = duckdb.connect(":memory:")

# Persistente
con = duckdb.connect("mi_db.duckdb")

# Con configuración
con = duckdb.connect(config={"threads": 4, "memory_limit": "8GB"})

# Resultado a distintos formatos
con.sql("SELECT ...").fetchall()   # lista de tuplas
con.sql("SELECT ...").df()         # Pandas DataFrame
con.sql("SELECT ...").pl()         # Polars DataFrame
con.sql("SELECT ...").arrow()      # PyArrow Table
```

## Leer archivos

```python
# Parquet
duckdb.sql("SELECT * FROM 'datos.parquet'")
duckdb.sql("SELECT * FROM read_parquet('carpeta/*.parquet')")
duckdb.sql("SELECT * FROM read_parquet('s3://bucket/**/*.parquet', hive_partitioning=true)")

# CSV
duckdb.sql("SELECT * FROM 'datos.csv'")
duckdb.sql("SELECT * FROM read_csv('datos.csv', delim=';', header=true)")

# JSON
duckdb.sql("SELECT * FROM read_json_auto('datos.json')")
duckdb.sql("SELECT * FROM 'datos.ndjson'")
```

## Escribir archivos

```python
duckdb.sql("COPY tabla TO 'salida.parquet' (FORMAT parquet, COMPRESSION zstd)")
duckdb.sql("COPY tabla TO 'salida.csv' (HEADER, DELIMITER ',')")
duckdb.sql("COPY tabla TO 'salida.json' (FORMAT JSON, ARRAY true)")
```

## Integración con DataFrames (zero-copy)

DuckDB detecta automáticamente variables DataFrame en el scope de Python:

```python
import pandas as pd
import polars as pl

df_pandas = pd.read_csv("datos.csv")
resultado = duckdb.sql("SELECT categoria, AVG(valor) FROM df_pandas GROUP BY 1").df()

df_polars = pl.read_parquet("datos.parquet")
resultado = duckdb.sql("SELECT * FROM df_polars WHERE valor > 100").pl()

# Registrar explícitamente (para threads o casos ambiguos)
con.register("mi_vista", df_pandas)
```

## SQL analítico destacado

```sql
-- Window functions
SELECT user_id,
       SUM(amount) OVER (PARTITION BY user_id ORDER BY ts ROWS 7 PRECEDING) AS rolling_7d
FROM eventos;

-- PIVOT
PIVOT ventas ON region USING SUM(monto) GROUP BY fecha;

-- Lambdas en SQL
SELECT list_transform([1,2,3], x -> x * 2);

-- ASOF joins (series de tiempo)
SELECT * FROM precios ASOF JOIN eventos USING (ts);
```

## Vector Search (extensión VSS)

```python
con.sql("INSTALL vss; LOAD vss")

con.sql("""
    CREATE TABLE docs (id INT, texto VARCHAR, embedding FLOAT[384])
""")
con.sql("CREATE INDEX idx ON docs USING HNSW (embedding)")

# Búsqueda por similitud coseno
resultados = con.execute("""
    SELECT texto, array_cosine_distance(embedding, ?) AS dist
    FROM docs ORDER BY dist ASC LIMIT 5
""", [query_embedding]).fetchall()
```

## UDFs

```python
# Escalar Python
con.create_function("c2f", lambda c: c * 9/5 + 32, ["DOUBLE"], "DOUBLE")

# Vectorizado Arrow (mucho más rápido)
import pyarrow.compute as pc
con.create_function("discount", lambda p: pc.multiply(p, 0.85),
                    ["DOUBLE"], "DOUBLE", type="arrow")
```

## DuckDB vs SQLite

| | DuckDB | SQLite |
|---|---|---|
| Modelo | Columnar | Row-based |
| Optimizado para | OLAP / Analítica | OLTP / Transaccional |
| Formatos nativos | Parquet, CSV, JSON, S3, Arrow | Solo tablas propias |
| DataFrames | Zero-copy nativo | Manual |
| Escrituras concurrentes | No (un writer) | Sí (WAL) |

**Regla**: SQLite para estado de app con escrituras frecuentes; DuckDB para analizar y procesar datos en volumen.
