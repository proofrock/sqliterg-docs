# 🎞 Features

* Aligned to [**SQLite 3.44.0**](https://sqlite.org/releaselog/3\_44\_0.html) for bundled builds;
* A [**single executable file**](https://germ.gitbook.io/sqliterg/documentation/installation) (written in Rust);
* [Can be built](../building-and-testing.md#types-of-binaries) against the system's SQLite or embedding one;
* HTTP/JSON access, with [**client libraries**](../integrations/client-libraries.md) for convenience;
* Directly call `sqliterg` on a database (as above), many options available using a YAML companion file;
* [**In-memory DBs**](../documentation/running.md#file-based-and-in-memory) are supported;
* Serving of [**multiple databases**](https://germ.gitbook.io/sqliterg/documentation/configuration-file) in the same server instance;
* Named or positional parameters in SQL are supported;
* [**Batching**](https://germ.gitbook.io/sqliterg/documentation/requests#batch-parameter-values-for-a-statement) of multiple value sets for a single statement;
* All queries of a call are executed in a [**transaction**](https://germ.gitbook.io/sqliterg/documentation/requests);
* For each query/statement, specify if a failure should rollback the whole transaction, or the failure is [**limited**](https://germ.gitbook.io/sqliterg/documentation/errors#managed-errors) to that query;
* "[**Stored Statements**](https://germ.gitbook.io/sqliterg/documentation/stored-statements)": define SQL in the server, and call it from the client;
* "[**Macros**](https://germ.gitbook.io/sqliterg/documentation/macros)": lists of statements that can be executed at db creation, at startup, periodically or calling a web service;
* [**Backups**](https://germ.gitbook.io/sqliterg/documentation/backups), rotated and also runnable at db creation, at startup, periodically or calling a web service;
* [**CORS**](https://germ.gitbook.io/sqliterg/documentation/configuration-file#corsorigin) mode, configurable per-db;
* [**Journal Mode**](https://sqlite.org/wal.html) (e.g. WAL) can be configured;
* [**Embedded web server**](../documentation/web-server.md) to directly serve web pages that can access `sqliterg` without CORS;
* [**Quite fast**](performances.md)!
* Comprehensive [**test suite**](../building-and-testing.md#testing);
* [**Docker images**](https://germ.gitbook.io/sqliterg/documentation/installation/docker), for x86\_64 and arm64;
* Binaries are provided with a bundled SQLite "inside" them, or linked against the system's installed SQLite.

## Security Features

* [**Authentication**](../security.md#authentication) can be configured
  * on the client, either using HTTP Basic Authentication or specifying the credentials in the request;
  * on the server, either by specifying credentials (also with hashed passwords) or providing a query to look them up in the db itself;
  * customizable `Not Authorized` error code (if `401` is not optimal);
* A database can be opened in [**read-only mode**](../security.md#read-only-databases) (only queries will be allowed);
* It's possible to enforce using [**only stored statements**](../security.md#stored-statements-to-prevent-sql-injection), to avoid some forms of SQL injection and receiving SQL from the client altogether;
* [**CORS Allowed Origin**](../security.md#cors-allowed-origin) can be configured and enforced;
* It's possible to [**bind**](../security.md#binding-to-a-network-interface) to a network interface, to limit access.

## Design choices

* Very thin layer over SQLite. Errors and type translation, for example, are those provided by the SQLite driver;
* Doesn't include HTTPS, as this can be done easily (and much more securely) with a [reverse proxy](../security.md#use-a-reverse-proxy-if-going-on-the-internet).
