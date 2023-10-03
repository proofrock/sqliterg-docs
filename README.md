---
description: Query sqlite via http - and remote clients too!
---

# 🌱 Introduction & Credits

_This is a rewrite in Rust of_ [_ws4sqlite_](https://github.com/proofrock/ws4sqlite)_, 30-50% faster, 10x less memory used, more flexible in respect to sqlite support. It is not a direct rewrite, more like a "sane" (I hope)_ [_redesign_](https://github.com/proofrock/sqliterg/blob/main/CHANGES\_FROM\_WS4SQLITE.md)_._

**BEWARE**: This documentation is being migrated/adapted from ws4sqlite.

## 🌿 Introduction

[**`sqliterg`**](http://github.com/proofrock/sqliterg) is a server-side application that, applied to one or more SQLite files, allows to perform SQL queries and statements on them via REST (or better, JSON over HTTP).

Possible use cases are the ones where remote access to a sqlite db is useful/needed, for example a data layer for a remote application, possibly serverless or even called from a web page ([_after security considerations_](security.md) of course).

[Client libraries](client-libraries.md) are available, that will abstract the "raw" JSON-based communication. See [here](https://github.com/proofrock/ws4sqlite-client-jvm) for Java/JVM, [here](https://github.com/proofrock/ws4sqlite-client-go) for Go(lang); others will follow. For now, the libraries are the same as `ws4sqlite`, that are largely compatible with `sqliterg` as well.

As a quick example, after launching:

```bash
sqliterg --db mydatabase.db
```

It's then possible to make a POST call to `http://localhost:12321/mydatabase`, e.g. with the following body:

```json
{
    "transaction": [
        {
            "statement": "INSERT INTO TEST_TABLE (ID, VAL, VAL2) VALUES (:id, :val, :val2)",
            "values": { "id": 1, "val": "hello", "val2": null }
        },
        {
            "query": "SELECT * FROM TEST_TABLE"
        }
    ]
}
```

Obtaining an answer of:

```json
{
    "results": [
        {
            "success": true,
            "rowsUpdated": 1
        },
        {
            "success": true,
            "resultSet": [
                { "ID": 1, "VAL": "hello", "VAL2": null }
            ]
        }
    ]
}
```

## 🥇 Credits

Kindly supported by [JetBrains for Open Source development](https://jb.gg/OpenSourceSupport)
