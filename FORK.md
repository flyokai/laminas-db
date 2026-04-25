# FORK notes — `flyokai/laminas-db`

> User docs → [`README.md`](README.md) · Agent quick-ref → [`CLAUDE.md`](CLAUDE.md) · Agent deep dive → [`AGENTS.md`](AGENTS.md)

This package is a maintenance fork of [`laminas/laminas-db`](https://github.com/laminas/laminas-db). It is consumed by every Flyokai project as the synchronous foundation of the database layer; async behaviour is added by sibling packages [`flyokai/laminas-db-driver-amp`](../laminas-db-driver-amp/README.md) and [`flyokai/laminas-db-driver-async`](../laminas-db-driver-async/README.md).

## Upstream

- Repository: <https://github.com/laminas/laminas-db>
- Status: feature-complete, security-only maintenance per Laminas TSC.
- License: BSD-3-Clause (preserved in `LICENSE.md`)

## Why we forked

The Laminas team announced security-only maintenance mode for `laminas/laminas-db`. The Flyokai framework needs:

- A small set of low-risk patches to make the driver / connection / statement / result interfaces sub-classable for fiber-aware async drivers.
- Confidence that the package will keep moving under our control if the upstream Laminas tree stops releasing entirely.

Forking gives us that without changing the public API.

## What changed

This fork should be **functionally equivalent** to upstream from any consumer's perspective. The diff is intentionally small:

- Minor extension points (visibility / inheritance) so the async driver packages can subclass cleanly.
- Selected upstream PRs not yet merged.

## Compatibility

This fork **`replace`s** `laminas/laminas-db` in `composer.json` (set in the project root, not in this package). Other libraries declaring `require: laminas/laminas-db` resolve to this fork.

Public class signatures are unchanged. Code written against upstream `laminas-db` should run against this fork.

## Tracking upstream

Periodically:

```bash
git remote add upstream https://github.com/laminas/laminas-db.git
git fetch upstream
git rebase upstream/master   # or `git merge upstream/master`
```

If upstream ships a security advisory, we cherry-pick the fix and tag a new release.

## License

BSD-3-Clause — Copyright (c) Laminas Project. See `LICENSE.md`. The fork adds no additional restrictions.
