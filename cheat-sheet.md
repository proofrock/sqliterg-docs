# 🔦 Cheat Sheet

## Commandline

```bash
sqliterg 
    --bind-host 0.0.0.0 \         # Optional
    --port 12321 \                # Optional
    --db ~/file1.db \             # File-based db, cfg at ~/file1.yaml
    --mem-db mem1::~/mem1.yaml \  # Memory-based db, with cfg
    --mem-db mem2 \               # Memory-based db, with default cfg
    --serve-dir myDir             # Serve static resources from a filesystem directory
    --index-file                  # The file to use as index (starting point)
                                  #  when serving a static directory
```

## Configuration file

This is taken from `db_conf.template.yaml` in the repository.

```yaml
# Main endpoint for requests is http://<host>:<port>/<db_name>

# If present, "auth" defines the authentication for this database
auth:
  # Optional, by default 401. The error HTTP code to be returned if auth fails.
  authErrorCode: 499
  # Mandatory. Defines how the credentials are passed to the server.
  #   "INLINE" means that credentials are passed in the request
  #   "HTTP_BASIC" uses Basic Authentication (via the "Authorization: Basic" header)
  mode: INLINE
  # Only one among "byQuery" and "byCredentials" must be specified.
  # This query validates credentials against a query in the database, it must have
  #   two parameters named ":user" and ":password" and if the credentials are valid must
  #   return (at least) one row.
  byQuery: SELECT 1 FROM AUTH WHERE USER = :user AND PASS = :password
  # This is a list of valid credentials, "statically" specified. "user" is case-insensitive
  #   while either a plaintext "password" or a SHA-256 hashed "hashedPassword" must be supplied.
  byCredentials:
    - user: myUser1
      password: ciao
    - user: myUser2
      hashedPassword: b133a0c0e9bee3be20163d2ad31d6248db292aa6dcb1ee087a2aa50e0fc75ae2 # "ciao"
# Journal mode for the database. Optional, default is "WAL". Not validated to accommodate for
#   future versions of SQLite. From SQLite docs: DELETE | TRUNCATE | PERSIST | MEMORY | WAL | OFF
journalMode: WAL
# Database is read-only. This is set after startup macros or backup are performed, so they can
#   still modify the database. It's implemented using the query_only PRAGMA.
readOnly: false
# Instruct the web server to povide the CORS header and preflight system as needed.
corsOrigin: "*"
# A "map" of statements (or queries) that can be called from request or macros using '^'. If I
#   want to use Q1, for example, I should specify '"query": "^Q1"' in the request.
storedStatements:
  - id: Q1
    sql: SELECT * FROM TBL
  - id: Q2
    sql: CREATE TABLE IF NOT EXISTS AUTH (USER TEXT, PASS TEXT)
# If set, only a Stored Statement can be used in the requests. Useful to avoid SQL injection.
useOnlyStoredStatements: false
# A "map" of macros, that are named groups of statements (not queries) that can be run at db
#   creation, at startup, every /n/ minutes, or via a web request.
macros:
  # ID of the macro
  - id: M1
    # Allow to execute out of a transaction, e.g. for VACUUM
    disableTransaction: false
    # Which statements it must execute (in a transaction). Stored Statements references can be used.
    statements:
      - CREATE TABLE IF NOT EXISTS TBL (ID INT, VAL TEXT)
      - ^Q2
    # Control of execution. Mandatory. All the contents have defaults meaning "disabled".
    execution:
      # Executes if the database is created (file wasn't present or in-memory)
      onCreate: false
      # Executes at each startup. Implies onCreate; if both are specified the macro will be executed once.
      onStartup: false
      # Executes every /n/ minutes. First execution is /n/ minutes after startup.
      period: 1 # in minutes, <= 0: never
      # Exposes an endpoint to execute the macro. A token-based for of authentication is mandatory.
      # Endpoint is http://<host>:<port>/<db_name>/macro/<macro_id>
      webService:
        # Optional, by default 401. The error HTTP code to be returned if auth fails.
        authErrorCode: 499
        # Either a plaintext "authToken" or a SHA-256 hashed "hashedAuthToken" must be supplied.
        authToken: ciao
        hashedAuthToken: b133a0c0e9bee3be20163d2ad31d6248db292aa6dcb1ee087a2aa50e0fc75ae2
# Optional. Configuration of the backups. Backups can be run at startup, every /n/ minutes,
#   or via a web request.
backup:
  # Directory for the backups. Must exist; mandatory config.
  backupDir: backups/
  # Keeps only the last /n/ backup files. Mandatory.
  numFiles: 3
  # Control of execution. Mandatory. All the contents have defaults meaning "disabled".
  execution:
    # Executes if the database is created (file wasn't present or in-memory)
    onCreate: false
    # Executes at each startup. Implies onCreate; if both are specified the macro will be executed once.
    onStartup: false
    # Executes every /n/ minutes. First execution is /n/ minutes after startup.
    period: 1 # in minutes, <= 0: never
    # Exposes an endpoint to execute the backup. A token-based for of authentication is mandatory.
    # Endpoint is http://<host>:<port>/<db_name>/backup
    webService:
      # Optional, by default 401. The error HTTP code to be returned if auth fails.
      authErrorCode: 499
      # Either a plaintext "authToken" or a SHA-256 hashed "hashedAuthToken" must be supplied.
      authToken: ciao
      hashedAuthToken: b133a0c0e9bee3be20163d2ad31d6248db292aa6dcb1ee087a2aa50e0fc75ae2
```

## Request

### URL

```
http://localhost:12321/<dbId>
```

### Headers

```
Content-Type: application/json
// If auth.mode == HTTP_BASIC, the header for basic authentication:
Authorization: Basic bXlVc2VyMTpjaWFv
```

### Body

```json
{
    "credentials": { // Necessary if and only if auth.mode == INLINE
        "user": "myUser1",
        "password": "myCoolPassword"
    },
    "transaction": [
        {
            "query": "SELECT * FROM TEMP"
        },
        {
            "query": "SELECT * FROM TEMP WHERE ID = :id",
            "values": { "id": 1 }
        },
        {
            "statement": "INSERT INTO TEMP (ID, VAL) VALUES (0, 'ZERO')"
        },
        {
            "noFail": true,
            "statement": "INSERT INTO TEMP (ID, VAL) VALUES (:id, :val)",
            "values": { "id": 1, "val": "a" }
        },
        {
            "statement": "^Q2", // '^' + the ID of the Stored Statement
            "valuesBatch": [
                { "id": 2, "val": "b" },
                { "id": 3, "val": "c" }
            ]
        }
    ]
}
```

## Response

### General Error (`400`, `401`, `404`, `409`, `500`)

```json
{
    "reqIdx": 1, // 0-based index of the failed subrequest; -1 for general
    "message": "near \"SELECTS\": syntax error"
}
```

### Success (`200`)

```json
{
    "results": [
        {
            "success": true,
            "resultSet": [
                { "ID": 1, "VAL": "ONE" },
                { "ID": 4, "VAL": "FOUR" }
            ]
        },
        {
            "success": true,
            "resultSet": [
                { "ID": 1, "VAL": "ONE" }
            ]
        },
        {
            "success": true,
            "rowsUpdated": 1
        },
        {
            "success": false,
            "error": "UNIQUE constraint failed: TEMP.ID"
        },
        {
            "success": true,
            "rowsUpdatedBatch": [ 1, 1 ]
        }
    ]
}
```
