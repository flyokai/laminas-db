# flyokai/laminas-db

> User docs → [`README.md`](README.md) · Agent quick-ref → [`CLAUDE.md`](CLAUDE.md) · Agent deep dive → [`AGENTS.md`](AGENTS.md)

> Database abstraction layer for the Flyokai framework — fork of [`laminas/laminas-db`](https://github.com/laminas/laminas-db).

Provides the synchronous foundation: adapters, drivers, SQL builders, table gateways, and platform-specific dialect generation. Async behaviour is added on top by sibling packages — this fork keeps the upstream API intact.

> Use [`flyokai/laminas-db-driver-amp`](../laminas-db-driver-amp/README.md) (native AMPHP MySQL) or [`flyokai/laminas-db-driver-async`](../laminas-db-driver-async/README.md) (PDO/MySQLi worker pools) on top of this package for non-blocking I/O.

## Features

- **Adapters & drivers** — Pdo, Mysqli, Pgsql, Oci8, IbmDb2, Sqlsrv
- **SQL builders** — `Select`, `Insert`, `Update`, `Delete` with fluent, platform-agnostic API
- **Platform abstraction** — Mysql, Postgresql, Oracle, SqlServer, Sqlite, IbmDb2, Sql92
- **Predicate system** — Between, EqualTo, In, IsNull, Like, …
- **TableGateway** — `select()`, `insert()`, `update()`, `delete()` with a feature/plugin system
- **Result sets** — buffered or forward-only
- **Metadata** — schema introspection (`MysqlMetadata`, `PostgresqlMetadata`, …)

## Installation

```bash
composer require laminas/laminas-db
```

Composer's `replace` makes this fork resolve under the upstream name automatically inside Flyokai installs.

## Quick start

```php
use Laminas\Db\Adapter\Adapter;
use Laminas\Db\Sql\Sql;

$adapter = new Adapter([
    'driver'   => 'Pdo_Mysql',
    'database' => 'app',
    'username' => 'app',
    'password' => 'secret',
    'hostname' => 'localhost',
]);

$sql    = new Sql($adapter);
$select = $sql->select('users')
              ->where(['status' => 'active'])
              ->order('created DESC')
              ->limit(20);

$stmt = $sql->prepareStatementForSqlObject($select);
$rows = iterator_to_array($stmt->execute());
```

## Adapter

Coordinates driver + platform + query execution.

- Query modes: `QUERY_MODE_PREPARE` (parameterised) / `QUERY_MODE_EXECUTE` (direct)
- `ProfilerInterface` support for query profiling
- Factory methods: `createStatement()`, `createDriver()`, `createPlatform()`

## SQL builders

```php
use Laminas\Db\Sql\Sql;

$sql    = new Sql($adapter);

$insert = $sql->insert('users')
    ->columns(['email', 'name'])
    ->values(['a@b.com', 'Alice']);

$update = $sql->update('users')
    ->set(['status' => 'disabled'])
    ->where(['email' => 'a@b.com']);

$delete = $sql->delete('users')->where(['email' => 'a@b.com']);
```

`Sql::buildSqlString($sqlObject)` generates platform-aware SQL; `Sql::prepareStatementForSqlObject($sqlObject)` returns a prepared statement.

## TableGateway

```php
use Laminas\Db\TableGateway\TableGateway;

$users = new TableGateway('users', $adapter);

$users->insert(['email' => 'a@b.com', 'name' => 'Alice']);
$rows = $users->select(['status' => 'active']);
$users->update(['status' => 'inactive'], ['id' => 7]);
$users->delete(['id' => 7]);
```

A `FeatureSet` plugs in pre/post operation hooks (e.g. metadata introspection, hydrator-based mapping).

## Async integration points

Async packages extend this foundation by:

1. Implementing `DriverInterface` / `ConnectionInterface` non-blockingly.
2. Wrapping `Statement::execute()` and `Connection::execute()` for fiber suspension.
3. Using the `Feature` system in `TableGateway` for async lifecycle hooks.

See [`flyokai/laminas-db-driver-amp`](../laminas-db-driver-amp/README.md) and [`flyokai/laminas-db-driver-async`](../laminas-db-driver-async/README.md).

## Gotchas

- **Empty WHERE protection** — `Update` and `Delete` have `$emptyWhereProtection = true` by default. They refuse to run without a WHERE clause to prevent full-table operations.
- **Forward-only results** — default is streaming. Call `buffer()` on the result set to enable re-iteration.
- **Platform abstraction** — SQL objects don't generate SQL directly; that's delegated to platform classes via `Sql::buildSqlString()`.
- **Parameter naming** — driver-specific (PostgreSQL `$1, $2` vs MySQL `?`). Use `ParameterContainer` for binding.
- **No async in this package** — use the driver-* siblings.

## License

BSD-3-Clause — Copyright (c) Laminas Project. See `LICENSE.md`.

## See also

- [`flyokai/laminas-db-driver-amp`](../laminas-db-driver-amp/README.md) — async via `amphp/mysql`
- [`flyokai/laminas-db-driver-async`](../laminas-db-driver-async/README.md) — async via worker pools / `MYSQLI_ASYNC`
- [`flyokai/laminas-db-bulk-update`](../laminas-db-bulk-update/README.md) — bulk inserts, ID resolution, range chunking
- [`flyokai/zend-db-sql-insertmultiple`](../zend-db-sql-insertmultiple/README.md) — multi-row `INSERT VALUES`
- Upstream: <https://docs.laminas.dev/laminas-db/>
