# 🏃 Running sqliterg

Running `sqliterg` can be done via the commandline, and it's possible to specify its behaviour via commandline parameters.

### Databases, `id` and Config (Companion) Files

`sqliterg` behavior when serving databases is configured with a mix of commandline parameters and naming conventions. You specify directly on the commandline the databases to serve, either specifying the path of the file (for file-based databases) or the id of the in-memory database.

`sqliterg` will look for a ["companion" configuration file](configuration-file/) in YAML format:

* for file-based databases, it can either:
  * be specified manually, after the file name and a double colon `::`&#x20;
  * or inferred from the file name, as a file that have the same base name of the database, but with `.yaml` extension.
* for in-memory databases, it can be specified manually after the database id and a double colon `::`

The companion file is used to load the serving parameters for that database. If it is not present or specified, default values will be used.

For example, if you run `sqliterg` as such:

{% code lineNumbers="true" %}
```bash
sqliterg --db ~/file1.db --mem-db mem1::~/mem1.yaml --mem-db mem2
```
{% endcode %}

It will:

* serve a db from `~/file1.db`, creating it if absent, with an id of `file1`;
* look for `~/file1.yaml`, and - if present - load from there the configuration for this db;
* serve a db from memory, with an id of `mem1`;
* load its configuration from `~/mem1.yaml`;
* serve a db from memory, with an id of `mem2`, and default configuration.

## Commandline Parameters

This is a complete commandline for `sqliterg`:

{% code lineNumbers="true" %}
```bash
sqliterg \
  --bind-host 0.0.0.0 \
  --port 12321 \
  --db myFile.db \
  --mem-db myMemDb[::myMemDb.yaml] \
  --serve-dir myDir/
  --index-file index.html
```
{% endcode %}

Of course, the usual `--help` and `--version` are supported. Let's discuss the other commandline parameters one by one.

#### `--bind-host`

Host to bind to. Defaults to `0.0.0.0`, meaning to accept connections from any local network interface. Different values are possible to restrict connections to a particular subnet.

#### `--port`

Port to use for incoming network communication. Defaults to `12321`.

#### `--db`

Can be repeated.

Specifies one or more file paths to load and serve as SQLite db's. It will use the base name (without the `.db` suffix) as the id of the database, to use in the URL of the [request](the-web-services/the-query-web-service/requests.md) , and will look for a configuration/companion file in the same path, named `<id>.yaml`.

It is also possible to specify a companion file at a different path, by specifying it after a double colon (`::`). Example: `--db myFile.db::/another/path/myFileConfig.yaml`.

#### `--mem-db`

Can be repeated.

Specifies one or more id for in-memory databases. Optionally, it's possible to specify also the path of the configuration file, after a double colon (`::`).

#### `--serve-dir`

Specifies a directory to serve via the internal web server. See the [relevant docs page](web-server.md).

#### `--index-file`

When serving static assets (see [above](running.md#serve-dir)) and no file is specified in the URL, redirects to a specific index file.

Meaningful only if `--serve-dir` is specified.

## Output

`sqliterg` will parse the commandline and the (possible) [config files](configuration-file/), it will attempt to open and connect to the various databases, creating their respective files as needed. Then it will output a summary of all the configurations, like this:

{% code lineNumbers="true" %}
```
sqliterg vX.Y.Z. based on SQLite v3.41.2

- Database 'db1'
  - from file 'db1.db'
    - file not present, it will be created
  - companion file not found: assuming defaults
  - journal mode: WAL
- Database 'db2'
  - from file 'db2.db'
  - parsing companion file 'db_conf.template.yaml'
  - allowed CORS origin: *
  - journal mode: WAL
- serving directory: /my/web/contents
  - with index file: index.html
- Listening on 0.0.0.0:12321
```
{% endcode %}
