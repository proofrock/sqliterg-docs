---
description: Query sqlite via http - and remote clients too!
---

# 🌿 Introduction & Credits

> _This is a rewrite in Rust of_ [_ws4sqlite_](https://github.com/proofrock/ws4sqlite)_,_ [_30-50% faster_](features/performances.md)_, 10x less memory used,_ [_more flexible_](building-and-testing.md#types-of-binaries) _in respect to sqlite support. It is not a direct rewrite, more like a "sane" (I hope) redesign. You can read more about what's changed and how to migrate_ [_here_](features/migrating-from-ws4sqlite.md)_._

[**`sqliterg`**](http://github.com/proofrock/sqliterg) is a server-side application that, applied to one or more SQLite files, allows to perform SQL queries and statements on them via REST (or better, JSON over HTTP).

Possible use cases are the ones where remote access to a sqlite db is useful/needed, for example a data layer for a remote application, possibly serverless or even called from a web page ([_after security considerations_](security.md) of course).

[Client libraries](integrations/client-libraries.md) are available, that will abstract the "raw" JSON-based communication. See [here](https://github.com/proofrock/ws4sqlite-client-jvm) for Java/JVM, [here](https://github.com/proofrock/ws4sqlite-client-go) for Go(lang); others will follow. For now, the libraries are the same as `ws4sqlite`, that are largely compatible with `sqliterg` as well.

As a quick example, after launching:

{% code lineNumbers="true" %}
```bash
sqliterg --db mydatabase.db
```
{% endcode %}

It's then possible to make a POST call to `http://localhost:12321/mydatabase`, e.g. with the following body:

{% code lineNumbers="true" %}
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
{% endcode %}

Obtaining an answer of:

{% code lineNumbers="true" %}
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
{% endcode %}

## 🥇 Credits

Kindly supported by [JetBrains for Open Source development](https://jb.gg/OpenSourceSupport)
