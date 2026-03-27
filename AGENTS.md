# wtsergo/laminas-db

Fork of Laminas DB providing the base database abstraction layer. Does NOT contain async implementations — async is provided by sibling packages (`laminas-db-driver-amp`, `laminas-db-driver-async`).

## Core Architecture

### Adapter Layer

**Adapter** — main entry point coordinating driver, platform, and query execution:
- Query modes: `QUERY_MODE_PREPARE` (parameterized) / `QUERY_MODE_EXECUTE` (direct)
- `ProfilerInterface` support for query profiling
- Factory methods: `createStatement()`, `createDriver()`, `createPlatform()`

### Driver System

Six driver implementations in `Adapter/Driver/`:
- **Pdo** — universal PDO (MySQL, PostgreSQL, SQLite, etc.)
- **Mysqli** — native mysqli extension with SSL support
- **Pgsql** — native pgsql extension (supports `PGSQL_CONNECT_ASYNC`)
- **Oci8** — Oracle
- **IbmDb2** — IBM DB2
- **Sqlsrv** — SQL Server

**Key interfaces:**
- `DriverInterface` — abstract driver operations
- `ConnectionInterface` — connect, disconnect, execute, transactions
- `StatementInterface` — prepared statement execution, parameter binding
- `ResultInterface` — result iteration (extends Iterator, Countable)

### SQL Abstraction Layer

Query builders in `Sql/`:
- **Select** — joins, GROUP BY, HAVING, ORDER, LIMIT, DISTINCT, FOR UPDATE, UNION
- **Insert** — single/bulk, INSERT...SELECT, VALUES_SET/VALUES_MERGE modes
- **Update** — with JOIN support, SET clause via PriorityList
- **Delete** — with WHERE filtering

**Sql** factory class:
- `buildSqlString($sqlObject)` — converts to platform-specific SQL
- `prepareStatementForSqlObject($sqlObject)` — creates prepared statement

**Platform classes** (`Sql/Platform/`): Mysql, Postgresql, Oracle, SqlServer, Sqlite, IbmDb2, Sql92 — handle identifier quoting, value escaping, dialect-specific syntax.

**Predicate system** (`Sql/Predicate/`): PredicateSet, Between, EqualTo, GreaterThan, In, IsNull, Like, etc.

### TableGateway Pattern

High-level ORM-like layer:
- `AbstractTableGateway` — `select()`, `insert()`, `update()`, `delete()`
- Feature/plugin system: `FeatureSet` with pre/post-operation hooks
- `HydratingResultSet` — hydrator integration for object mapping

### Result Set Handling

- `AbstractResultSet` — buffering (eager/lazy), Iterator interface
- `ResultSetInterface` — extends Iterator, Countable
- `buffer()` to force-load for re-iteration

### Metadata

Schema introspection: `MysqlMetadata`, `PostgresqlMetadata`, etc.
Object representations: `TableObject`, `ColumnObject`, `ConstraintObject`.

## Async Integration Points

This package provides the foundation. External async packages extend by:
1. Implementing `DriverInterface` / `ConnectionInterface` with non-blocking operations
2. Wrapping `Statement.execute()` and `Connection.execute()` for fiber suspension
3. Using the Feature system in TableGateway for async lifecycle hooks

## Gotchas

- **Empty WHERE protection**: Update and Delete have `$emptyWhereProtection = true` by default — prevents accidental full-table operations.
- **Forward-only results**: Default results are streaming. Call `buffer()` to enable re-iteration.
- **Platform abstraction**: SQL objects don't generate SQL directly — delegated to Platform classes via `Sql::buildSqlString()`.
- **Parameter containerization**: Driver-specific naming (PostgreSQL `$1,$2` vs MySQL `?`). Use `ParameterContainer` for binding.
- **No async in this package**: All async behavior comes from `laminas-db-driver-amp` and `laminas-db-driver-async`.
