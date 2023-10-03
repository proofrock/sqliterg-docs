# 📃 Macros

Macros are named groups of statements that can be run in a transaction or out of it, together.

The relevant piece of [configuration](configuration-file/) is:

{% code lineNumbers="true" %}
```yaml
macros:
  - id: M1
    disableTransaction: false
    statements:
      - CREATE TABLE IF NOT EXISTS TBL (ID INT, VAL TEXT)
      - ^Q2
    execution:
      onCreate: false
      onStartup: false
      period: 1 
      webService:
        authErrorCode: 499
        authToken: ciao
        # or hashedAuthToken: b133a0c0e9bee3be20163d...
```
{% endcode %}

This specifies a list of macros, with an id, a set of statements and an execution policy that tells `sqliterg` when to execute it.&#x20;

#### `id`

_Line 2, string, mandatory_

The id of the macro.

#### `disableTransaction`

_Line 3, boolean_

When executing certain statements (most notable, `VACUUM`), SQLite requires not to use a transaction. When this parameter is set to `true`, a transaction is not used.

#### `statements`

_Lines 4-6, list of strings_

The list of statements to execute, in order. `^` syntax can be used to refer to [Stored Statements](configuration-file/stored-statements.md).

Statements are executed in a transaction, unless `disableTransaction` is set. If more than one macro is set to be executed at a certain stage (e.g. at [startup](macros.md#greater-than-onstartup)), each macro will have its own transaction.

{% hint style="warning" %}
Queries (that return a result set) cannot be specified in a macro.
{% endhint %}

#### `execution` policy

_Lines 7-14, object_

This object sets the means of execution of a macro. Within it are:

#### >`onCreate`

_Line 8, boolean_

If `true`, the macro will be executed at database creation, i.e. when

* a database file is configured, but does not exist at startup;
* an in-memory database is configured.

It's useful to initialize tables or other structures in a database.

If an error occurs in the macro of a file-based database, the new file is deleted.

#### >`onStartup`

_Line 9, boolean_

If `true`, the macro will be executed at database startup, i.e. when `sqliterg` starts.

If an error occurs in the macro of a file-based database, and the file was not present when the application was started up, the new file is deleted.

{% hint style="info" %}
If a macro is configured for both `onCreate` and `onStartup` and the database is new, the macro will be performed only once.
{% endhint %}

#### >`period`

_Line 10, number_

The macro can be configured to be run periodically. If this configuration is present and > 0, the macro will be executed every _n_ minutes, where _n_ is the period configured.

The first execution will be after _n_ minutes from the startup, then after _2n_ and so on.

#### >`webService`

_Lines 11-14_

Last, but absolutely not least, it's possible to call a web service to execute a macro.

It's callable via POST or GET, at the following URL:

{% code lineNumbers="true" %}
```url
http://<host>:<port>/<db_name>/macro/<macro_id>
```
{% endcode %}

The endpoint can be authenticated with a token-based setup. See the [relevant documentation](the-web-services/authentication.md#token-based-auth-for-macros-and-backup) for more info.
