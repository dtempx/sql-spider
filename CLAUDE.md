# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- **Package Manager**: This project uses **yarn** (not npm). Use `yarn install` / `yarn add` — do not create a `package-lock.json`.
- Build: `npx tsc` - Compiles TypeScript using tsc
- Clean: `./clean.sh` - Removes compiled JS, map, and declaration files
- Test: `npx mocha` - Runs tests using Mocha (test files use .test.ts extension)

## Architecture Overview
This is a multi-database data connector library that provides a unified interface for querying PostgreSQL, MySQL, SQLite, SQL Server, BigQuery, and Snowflake. The codebase follows a modular structure:

- **Main Entry Point**: `index.ts` re-exports everything from `lib/index.js`
- **Core Modules**: 
  - `lib/bigquery.ts` - BigQuery client with query/insert operations and data formatting
  - `lib/snowflake.ts` - Snowflake client with connection pooling, query/execute/insert operations
  - `lib/postgres.ts` - PostgreSQL client (node `pg` pool), also drives Postgres-compatible databases
  - `lib/mysql.ts` - MySQL client (`mysql2` pool), also drives MySQL-compatible databases like MariaDB
  - `lib/mssql.ts` - Microsoft SQL Server client (`mssql` pool)
  - `lib/sqlite.ts` - SQLite client (`better-sqlite3`, file-based or in-memory)
  - `lib/connect.ts` - Runtime connector selection by name (`connect("mysql", ...)`)
  - `lib/utilities.ts` - Shared utility functions and the `BaseConnector` surface

## Key Technical Details
- **Environment Variables**: Environment variables are used to hold connection strings
- **Query Parameters**: Both connectors support parameterized queries with different binding formats
- **Data Formatting**: Both connectors normalize result formats (BigQuery dates/timestamps, Snowflake lowercase keys)
- **Verbose Logging**: Set `VERBOSE=1` to enable query logging and timing information
- **Type Safety**: Uses TypeScript with strict mode, ESNext target, and ES modules

## Code Style Guidelines
- Use ES module imports (`import x from 'y'`)
- TypeScript with strict mode enabled
- Use camelCase for variables/functions, PascalCase for classes/interfaces
- Error handling with try/catch for async operations
- Keep code self-explanatory with minimal commenting