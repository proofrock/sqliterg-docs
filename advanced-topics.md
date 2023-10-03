# 🎓 Advanced Topics

## Concurrency (or lack thereof)

All transactions are submitted to the database on one connection, in non-concurrent fashion, i.e. at a given time at most one transaction can be active. A per-database mutex takes care of this.

If two databases are opened on the same file (see next point) there could be concurrent accesses, so take care to set WAL etc. accordingly. Also, it's your duty to check and retry on SQL\_BUSY errors.

## Serve two instances of the same database file

It can be done, and it makes perfect sense (e.g. to expose a read only instance without authentication, and a read-write one with auth). Just specify the same file in two db configurations, and use them normally.

See FAQ #5 at the [SQLite's site](https://www.sqlite.org/faq.html). Again, it's your duty to check and retry on possible SQL\_BUSY errors.
