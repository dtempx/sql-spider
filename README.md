# SQL-Spider

A simple data adapter that provides a single, consistent interface for querying across PostgreSQL, MySQL, SQLite, Snowflake, BigQuery, and SQL Server.

Instead of juggling different SDKs and connection patterns--sql-spider abstracts away the complexity so you can focus on your data.

## Supported Databases
- **PostgreSQL** - Open-source relational database
- **MySQL** - The world's most popular open-source relational database (also drives MySQL-compatible databases like MariaDB)
- **SQLite** - Embedded, file-based (or in-memory) SQL database
- **Snowflake** - Cloud-native data platform
- **BigQuery** - Google Cloud's serverless data warehouse
- **Microsoft SQL Server** - Microsoft's relational database engine

## Why sql-spider?
Instead of learning different APIs for each data warehouse:

```javascript
// Without sql-spider - different patterns for each warehouse
import { BigQuery } from '@google-cloud/bigquery';
import snowflake from 'snowflake-sdk';

// BigQuery setup
const bigquery = new BigQuery({ projectId: 'my-project' });
const [rows] = await bigquery.query({ query: 'SELECT ...' });

// Snowflake setup  
const connection = snowflake.createConnection({ ... });
connection.connect();
connection.execute({ sqlText: 'SELECT ...', complete: callback });
```

Use one simple, consistent interface:

```javascript
// With sql-spider - same pattern everywhere
import { bigquery, snowflake, sqlite } from "sql-spider";

const r1 = await bigquery.query("SELECT ...");
const r2 = await snowflake.query("SELECT ...");
const r3 = await sqlite.query("SELECT ...");
```

## Key Features
- **Unified Data Interface**: Same query interface across all supported databases
- **Parameterized Queries**: Supports safe parameter binding
- **Data Normalization**: Consistent row format result across databases (array of javascript objects)
- **Runtime Connector Abstraction**: Decide at runtime which database environment to connect to

## Installation
```bash
npm install sql-spider
# or
yarn add sql-spider
```

## Quick Start

### Environment Setup

Each connector's default instance reads its connection info from an environment variable. The formats for every connector are documented in [Environment Variables](docs/environment-variables.md).

**BigQuery**: Uses Google Cloud default credentials or service account key
**Snowflake**: Set connection details in `SNOWFLAKE_CONNECTION` environment variable:
```bash
export SNOWFLAKE_CONNECTION="account:myaccount,username:myuser,password:mypass,database:mydb,warehouse:mywh"
```
**PostgreSQL**: Set the connection string in the `POSTGRES_CONNECTION` environment variable:
```bash
export POSTGRES_CONNECTION="postgres://myuser:mypass@localhost:5432/mydb"
```
**MySQL**: Set the connection string in the `MYSQL_CONNECTION` environment variable:
```bash
export MYSQL_CONNECTION="mysql://myuser:mypass@localhost:3306/mydb"
```
**Microsoft SQL Server**: Set the connection string in the `MSSQL_CONNECTION` environment variable:
```bash
export MSSQL_CONNECTION="Server=localhost,1433;Database=mydb;User Id=myuser;Password=mypass;Encrypt=true;TrustServerCertificate=true"
```
**SQLite**: Set the database file path in `SQLITE_CONNECTION` (defaults to an in-memory database when unset):
```bash
export SQLITE_CONNECTION="./data.db"   # or ":memory:" for an ephemeral in-memory database
```

## BigQuery example
```javascript
import { bigquery } from "sql-spider";

const sql = "SELECT word, COUNT(*) as word_count FROM bigquery-public-data.samples.shakespeare GROUP BY ALL ORDER BY 2 DESC LIMIT 10";

const rows = await bigquery.query(sql);
for (const row of rows)
    console.log(JSON.stringify(row));
```


## Snowflake example
```javascript
import { snowflake } from "sql-spider";

const sql = "SELECT table_schema, table_name, table_type FROM INFORMATION_SCHEMA.TABLES WHERE table_schema != 'INFORMATION_SCHEMA' LIMIT 10";

const rows = await snowflake.query(sql);
for (const row of rows)
    console.log(JSON.stringify(row));
snowflake.close();
```

## PostgreSQL example
```javascript
import { postgres } from "sql-spider";

await postgres.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)");
await postgres.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await postgres.query("SELECT * FROM users WHERE id = $1", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
postgres.close();
```

> The `postgres` connector also works with databases that speak the PostgreSQL wire protocol — CockroachDB, Redshift, YugabyteDB, AlloyDB, TimescaleDB — by pointing it at a different connection string. [more info](docs/postgres-compatible-databases.md)

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

> The `mysql` connector also works with databases that speak the MySQL wire protocol, such as MariaDB.

## SQLite example
```javascript
import { sqlite } from "sql-spider";

await sqlite.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER, name TEXT)");
await sqlite.insert("users", [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }]);

const rows = await sqlite.query("SELECT * FROM users WHERE id = ?", [1]);
for (const row of rows)
    console.log(JSON.stringify(row));
```

## Connector Abstraction
The `connect` function enables the creation of an *abstract* database connection, where the *concrete* backend behind it is decided while the program is running—not fixed in the source by an `import`. [more info](docs/connector-abstraction.md)

## Multi-Instance Connectors
Need to pass connection info at runtime, pick a backend from config, or talk to more than one instance of the same warehouse (e.g. two Snowflake warehouses)? [more info](docs/multi-instance.md)

## Alternatives
How does sql-spider compare to other alternatives and where does it fit? [more info](docs/alternatives.md)

## License
MIT
