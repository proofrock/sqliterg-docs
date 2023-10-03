# 📃 Configuration File(s)

The configuration files are a set of companion files for each databases. They can be absent, and the database will take default values; if they are present, they indicate the parameters with which `sqliterg` must "treat" the database.

For **file based db**s, the configuration file follows a naming convention: it must have the same path and base filename (i.e. the filename without the extension) of the db file, buit with a `.yaml` extension added. Example: `file.db` → `file.yaml`.&#x20;

For **memory based db**s, the config files can be specified in the [`--mem-db`](../running.md#mem-db) commandline argument. More details [here](../running.md#databases-id-and-config-companion-files).

The configuration is in YAML format. A couple of examples that describe the entire set of configurations are as follows. A complete [cheat sheet](../../cheat-sheet.md#configuration-file) is available.

{% code title="Example #1" lineNumbers="true" %}
```yaml
auth:
  authErrorCode: 499
  mode: HTTP_BASIC
  byCredentials:
    - user: myUser1
      password: myCoolPassword
    - user: myUser2
      hashedPassword: b133a0c0e9bee3be20163d...
journalMode: WAL
readOnly: false
corsOrigin: "*"
backup:
  backupDir: backups/
  numFiles: 3
  execution:
    onCreate: false
    onStartup: false
    period: 1 # in minutes
    webService:
      authErrorCode: 499
      authToken: ciao
      hashedAuthToken: b133a0c0e9bee3be20163d...
```
{% endcode %}

{% code title="Example #2" lineNumbers="true" %}
```yaml
auth:
  mode: INLINE
  byQuery: SELECT 1 FROM AUTH WHERE USER = :user AND PASSWORD = :password
useOnlyStoredStatements: true
storedStatements:
  - id: Q1
    sql: SELECT * FROM TEMP 
  - id: Q2
    sql: INSERT INTO TEMP VALUES (:id, :val)
macros:
  - id: M1
    disableTransaction: false
    statements:
      - CREATE TABLE IF NOT EXISTS TBL (ID INT, VAL TEXT)
      - ^Q2
    execution:
      onCreate: false
      onStartup: false
      period: 1 # in minutes
      webService:
        authErrorCode: 499
        authToken: ciao
        # or hashedAuthToken: b133a0c0e9bee3be20163d...
```
{% endcode %}

If an error occurs while parsing the configuration, the application will exit with a status code of 1, after printing the error to `stderr`.

A description of the fields (or areas) follows, with indication of the relevant lines in the examples.

#### `auth`

_Lines 1-8 of #1, 1-3 of #2; object_

Configuration for the authentication. See the [relevant section](../the-web-services/authentication.md).

#### `journalMode`

_Line 9 of #1; boolean_

By default a database is opened in [WAL mode](https://sqlite.org/wal.html). This line instructs `sqliterg` to open the database using another journal mode.&#x20;

This parameter is not validated, so that future versions of SQLite can be used (e.g. with WAL2).

Possible values are `DELETE`, `TRUNCATE`, `PERSIST`, `MEMORY`, `WAL` and `OFF` as per SQLite's documentation.

Under the hood, this is performed by using the `journal_mode=WAL` pragma.

#### `readOnly`

_Line 10 of #1; boolean_

If this flag is present and set to `true`, the database will be treated as read-only. Only queries are allowed, that don't alter the database structure.

Please note that macros performed at creation or startup are still allowed to change the database.

Under the hood, this is performed by using the `query_only=true` pragma.

#### `corsOrigin`

_Line 11 of #1; string_

If specified, it enables serving CORS headers in the response, and specifies the value of the `Access-Control-Allow-Origin` header.

It can be set to `*`. In this case, beware to put double quotes (`"*"`) as the asterisk is a special character for YAML.

Used to both configure the server to serve CORS headers and, when non-`*`, restrict access to calls from a single, trusted web address.

#### `backup`

_Lines 12-22 of #1; object_

Configuration of the backups. Backups can be run at startup, every _n_ minutes, or via a web request.

See the [relevant section](./#backup).

#### `storedStatements` and `useOnlyStoredStatements`

_`storedStatements`: Lines 5-9 of #2; object_\
_`useOnlyStoredStatements`: Line 4 of #2; boolean_

Stored Statements are a way to specify (some of) the statement/queries you will use in the server instead of sending them over from the client.

You can also instruct `sqliterg` to allow only usage of stored statements, so that the client calls cannot contain SQL, for [security reasons](../../security.md#stored-statements-to-prevent-sql-injection).

See the [relevant section](stored-statements.md).

#### `macros`

_Lines 10-23 of #2; list of objects_

Macros are named groups of statements (as in: not queries) that can be run at db creation, at startup, every /n/ minutes, or via a web request.

See the [relevant section](../macros.md).
