# SQL-Spider Alternatives

If you're weighing sql-spider against the popular options, here's how it fits:

| Library | Space | Supported Databases | Style |
|---|---|---|---|
| [Knex.js](https://knexjs.org/) | OLTP query builder | Postgres, MySQL, SQLite | Chainable query builder |
| [Kysely](https://kysely.dev/) | Type-safe query builder | Postgres, MySQL, SQLite | Chainable, strong TS inference |
| [Prisma](https://www.prisma.io/) / [TypeORM](https://typeorm.io/) / [Sequelize](https://sequelize.org/) | ORM | Postgres, MySQL, SQLite | Models, relations, migrations |
| **sql-spider** | Warehouse + OLTP + embedded adapter | Postgres, MySQL, SQLite, Snowflake, BigQuery, SQL Server | Thin raw-SQL pass-through |

**Knex** and **Kysely** are the most popular libraries for "write once, swap the backend." They're the right choice if you want a query *builder* over traditional relational databases (Postgres, MySQL, SQLite, MSSQL). However, they do **not** target cloud data warehouses.

The **ORMs** (Prisma, TypeORM, Sequelize) add models, relations, and migrations on top — a heavier abstraction than a query interface, and likewise aimed at OLTP databases.

**sql-spider fills a different niche:** a lightweight, raw-SQL, unified interface that spans cloud data *warehouses* (BigQuery, Snowflake) alongside traditional OLTP databases (PostgreSQL, MySQL, SQL Server) and embedded SQLite. If you want to run SQL across those backends behind one small API — rather than build queries or model relations — that's what this library is for.
