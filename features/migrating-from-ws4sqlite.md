# 🕊 Migrating from ws4sqlite

Here is a list of changes that need to be done or be aware of when migrating from ws4sqlite.

## Changes to the project

* License is now Apache v2;
* Project is written in Rust, with a profiler written in Java+Bash and a test suite in Go;

## Changes to your code

* The separator between filename (or database Id in the case of in-memory databases) and YAML file is now `::` instead of a single `:`. See [here](../documentation/running.md#databases-id-and-config-companion-files).
* To refer to a Stored Statement, the `^id` syntax must be used, not `#id`. See [here](../documentation/configuration-file/stored-statements.md#usage-in-requests).
* In the queries or statements, parameters cannot be written in the SQL using `@param` anymore; `:param` must be used. See [here](../documentation/the-web-services/the-query-web-service/requests.md#parameter-values-for-the-query-statement).
* It's not possible to use `values` and `valuesBatch` in the same request anymore. See [here](../documentation/the-web-services/the-query-web-service/requests.md#batch-parameter-values-for-a-statement).
* `HTTP` authentication is now named `HTTP_BASIC` (possible future JWT implementation). See [here](../documentation/the-web-services/authentication.md#mode).
* `disableWALMode` is now named as a more generic `journalMode`, for flexibility. See [here](../documentation/configuration-file/#journalmode).
* Encryption feature is not available anymore. It was effective and standard-based, but too "vertical". Use client-based encryption, if needed.
* ScheduledTasks and InitStatements are replaced with the new [macro](../documentation/macros.md) and [backup](../documentation/backup.md) subsystems, that should be more flexible.
  * There's not a cron-like scheduler anymore; it was very language-dependent. You can use a macro/backup [webService](../documentation/the-web-services/#ws-for-macros-configurable) with curl and a system crontab.
* The backup files' pattern is specified as a backup directory, being intended that all the files in it are subject to deletion. See [here](../documentation/backup.md#backupdir).
* It is possible to specify the index file when serving a directory. See [here](../documentation/running.md#index-file).

## Other changes

* Read-only mode is performed via the `query_only` PRAGMA;
* Even if the database is read only, it's still possible to perform [onCreate](../documentation/macros.md#greater-than-oncreate)/[onStartup](../documentation/macros.md#greater-than-onstartup) macros;
* When an auth error occurs, both `ws4sqlite` and `sqliterg` block the request for 1 second. The difference is that `ws4sqlite` blocks also all the other requests on the same database, while `sqliterg` does not. `ws4sqlite`'s behavior is more prone to DDOS attacks.
