# Dolt — Git para Datos

Base de datos SQL con versionado tipo Git en el storage layer. Wire protocol MySQL-compatible.

## Instalación

```bash
brew install dolt
dolt config --global --add user.email "tu@email.com"
dolt config --global --add user.name "Tu Nombre"
```

## Inicializar y arrancar

```bash
mkdir mi-db && cd mi-db
dolt init
dolt sql-server --port 3306      # expone servidor MySQL-compatible
```

## CLI — mapeo Git → Dolt

```bash
dolt init
dolt add -A && dolt commit -m "mensaje"
dolt branch feat && dolt checkout feat
dolt merge feat
dolt diff                        # diff de datos
dolt log                         # historial de commits
dolt clone owner/repo            # desde DoltHub
dolt push / dolt pull
```

## Versioning desde SQL

```sql
-- Escrituras de versioning (stored procedures)
CALL DOLT_ADD('-A');
CALL DOLT_COMMIT('-Am', 'mensaje');
CALL DOLT_BRANCH('feature-v2');
CALL DOLT_CHECKOUT('feature-v2');
CALL DOLT_MERGE('feature-v2');

-- Lectura de historial (system tables)
SELECT * FROM dolt_log;
SELECT * FROM dolt_diff('main', 'feature-v2', 'mi_tabla');
SELECT * FROM tabla AS OF 'abc1234';  -- estado en un commit específico
```

## Python (recomendado: mysql-connector)

```python
import mysql.connector

# Conectarse a un branch específico: "database/branch"
cnx = mysql.connector.connect(
    user='root', password='', host='127.0.0.1',
    database='midb/main'
)
cursor = cnx.cursor()
cursor.execute("INSERT INTO tabla VALUES (...)")
cursor.callproc("DOLT_COMMIT", ("-Am", "Agrego datos"))
cursor.execute("SELECT * FROM dolt_log")
```

## Python con doltcli

```python
from doltcli import Dolt
db = Dolt.init("/ruta/a/db")
db.sql("INSERT INTO tabla VALUES (...)")
db.add("tabla")
db.commit("Nuevo batch de datos")
```

> **Gotcha**: `doltcli` requiere Python ≤ 3.12 (`typed-ast` no compila en 3.13+).

## SQLAlchemy

```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://root:@localhost/midb")
# Toda la API de SQLAlchemy funciona igual que con MySQL
```

## Casos de uso para AI/ML

- **Reproducibilidad**: commit del dataset → `SELECT * FROM tabla AS OF 'sha'` recupera exactamente esos datos
- **Agentic workflows**: agent escribe en branch aislado → humano revisa diff → merge a main
- **Feature stores versionados**: branch `main` para producción, branches para experimentos
- **Vectores versionados**: soporte de vectores desde 2025, útil para RAG con embeddings reproducibles

## DoltHub

```bash
dolt clone dolthub/us-jails      # clonar dataset público
dolt push origin main            # pushear a DoltHub
```
