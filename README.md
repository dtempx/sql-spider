# SQL-Spider

A simple data adapter that provides a single, consistent interface for querying across PostgreSQL, MySQL, SQLite, DuckDB, Snowflake, BigQuery, and SQL Server.

Instead of juggling different SDKs and connection patterns, SQL-Spider abstracts away the complexity so you can focus on your data.

## Supported Databases
- **PostgreSQL** - Open-source relational database
- **MySQL** - Open-source relational database
- **Microsoft SQL Server** - Microsoft proprietary relational database engine
- **SQLite** - Embedded, file-based (or in-memory) SQL database
- **DuckDB** - Embedded, file-based (or in-memory) analytical (OLAP) database
- **Snowflake** - Multi-cloud data warehouse *(runs on AWS, Azure, or GCP)*
- **BigQuery** - Google Cloud's serverless data warehouse

> Postgres also works with databases that speak the PostgreSQL wire protocol, namelyCockroachDB, Redshift, YugabyteDB, AlloyDB, TimescaleDB. [Learn more](docs/postgres-compatible-databases.md)

> MySQL also works with MySQL-compatible databases like MariaDB.

## Why SQL-Spider?
Instead of learning different APIs for each database engine:

```javascript
// Without SQL-Spider - different patterns for each database engine
import pg from 'pg';
import mysql from 'mysql2/promise';

// Postgres setup
const pool = new pg.Pool({ connectionString: 'postgres://...' });
const { rows } = await pool.query('SELECT ...');

// MySQL setup
const connection = await mysql.createConnection({ host: '...', user: '...' });
const [rows] = await connection.execute('SELECT ...');

// SQL Server setup
import mssql from 'mssql';
const pool = await mssql.connect('Server=host,1433;Database=db;User Id=sa;Password=pw');
const { recordset } = await pool.request().query('SELECT ...');
```

Use one simple, consistent interface:

```javascript
// With SQL-Spider - same pattern everywhere
import { postgres, mysql, mssql } from "sql-spider";

const r1 = await postgres.query("SELECT ...");
const r2 = await mysql.query("SELECT ...");
const r3 = await mssql.query("SELECT ...");
```

## Key Features
- **Unified Data Interface**: Same query interface across all supported databases.
- **Data Normalization**: Consistent row format result across databases *(returns an array of javascript objects)*.
- **Runtime Connector Abstraction**: Decide at runtime which database environment to connect to.
- **Parameterized Queries**: Supports safe parameter binding—positional and/or named depending on the database. [Learn more](docs/query-parameters.md)

## Installation
```bash
npm install sql-spider
# or
yarn add sql-spider
```

## Quick Start
Each connector's default instance reads its connection info from an environment variable. The formats for every connector are documented in [Environment Variables](docs/environment-variables.md).

## Postgres example
```javascript
import { postgres } from "sql-spider";

await postgres.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)");
await postgres.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await postgres.query("SELECT * FROM users WHERE id = $1", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
postgres.close();
```

> Set the `POSTGRES_CONNECTION` environment variable to a value like `postgres://myuser:mypass@localhost:5432/mydb`.

## MySQL example
```javascript
import { mysql } from "sql-spider";

await mysql.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)");
await mysql.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await mysql.query("SELECT * FROM users WHERE id = ?", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
mysql.close();
```

> Set the `MYSQL_CONNECTION` environment variable to a value like `mysql://myuser:mypass@localhost:3306/mydb`.

## SQL Server example
```javascript
import { mssql } from "sql-spider";

await mssql.execute("CREATE TABLE users (id INTEGER, name VARCHAR(255))");
await mssql.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await mssql.query("SELECT * FROM users WHERE id = @p0", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
mssql.close();
```

> Set the `MSSQL_CONNECTION` environment variable to a value like `Server=localhost,1433;Database=mydb;User Id=myuser;Password=mypass;Encrypt=true` or `mssql://myuser:mypass@localhost:1433/mydb`.

## SQLite example
```javascript
import { sqlite } from "sql-spider";

await sqlite.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)");
await sqlite.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await sqlite.query("SELECT * FROM users WHERE id = ?", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
```

> Set the `SQLITE_CONNECTION` environment variable to a value that specifies the path to a local file, or leave unspecified and it will default to an in-memory database.

## DuckDB example
```javascript
import { duckdb } from "sql-spider";

await duckdb.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)");
await duckdb.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await duckdb.query("SELECT * FROM users WHERE id = ?", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
duckdb.close();
```

> Set the `DUCKDB_CONNECTION` environment variable to a value that specifies the path to a local file, or leave unspecified and it will default to an in-memory database.

## Snowflake example
```javascript
import { snowflake } from "sql-spider";

const sql = "SELECT table_schema, table_name, table_type FROM INFORMATION_SCHEMA.TABLES WHERE table_schema != 'INFORMATION_SCHEMA' LIMIT 10";

const rows = await snowflake.query(sql);
for (const row of rows)
    console.log(JSON.stringify(row));
snowflake.close();
```

> Set the `SNOWFLAKE_CONNECTION` environment variable to a value like `account:myaccount,username:myuser,password:mypass,database:mydb,warehouse:mywh`.

## BigQuery example
```javascript
import { bigquery } from "sql-spider";

const sql = "SELECT word, COUNT(*) as word_count FROM bigquery-public-data.samples.shakespeare GROUP BY ALL ORDER BY 2 DESC LIMIT 10";

const rows = await bigquery.query(sql);
for (const row of rows)
    console.log(JSON.stringify(row));
```

> Uses Google Cloud default credentials or a service account key specified in the `GOOGLE_APPLICATION_CREDENTIALS` environment variable. No connection string is required when running within Google Cloud.

## Local Databases
SQLite and DuckDB are *embedded* — the database is a file on disk (or in memory) rather than a server you connect to. [Learn more](docs/local-databases.md)

## Abstract `connect` Function
The `connect` function enables the creation of an *abstract* database connection, where the *concrete* backend behind it is decided while the program is running—not fixed in the source by an `import`. [Learn more](docs/connector-abstraction.md)

## Multiple Connections to Same Type
Need to pass connection info at runtime, pick a backend from config, or talk to more than one instance of the same database (e.g. two Snowflake warehouses)? [Learn more](docs/multi-instance.md)

## Alternatives
How does SQL-Spider compare to other alternatives and where does it fit? [Learn more](docs/alternatives.md)

## License
MIT
