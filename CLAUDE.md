# flyokai/laminas-db

> User docs → [`README.md`](README.md) · Agent quick-ref → [`CLAUDE.md`](CLAUDE.md) · Agent deep dive → [`AGENTS.md`](AGENTS.md)

Fork of Laminas DB — base database abstraction layer. Async provided by sibling driver packages.

See [AGENTS.md](AGENTS.md) for detailed documentation.

## Quick Reference

- **Adapter**: Coordinates driver + platform + query execution
- **Drivers**: Pdo, Mysqli, Pgsql, Oci8, IbmDb2, Sqlsrv
- **SQL builders**: Select, Insert, Update, Delete — fluent API, platform-agnostic
- **TableGateway**: ORM-like `select()`/`insert()`/`update()`/`delete()` with feature plugins
- **Platforms**: Mysql, Postgresql, Oracle, SqlServer, Sqlite — dialect-specific SQL generation
- **Key rule**: No async here — use `laminas-db-driver-amp` or `laminas-db-driver-async` for non-blocking
