# 📼 Backup

Backups are what you expect: `sqliterg` backs up the SQLite database, to a backup folder, rotating after it reaches a certain number of backup files.

Backups can be triggered in a variety of ways, and are performed using the `VACUUM INTO` SQL command of SQLite.

## Naming

The file name is composed from the database filename or base name, adding to it a timestamp in the format:

```
_YYYYMMDD-HHMM    // 24h format
```

More precisely:

* For file-based databases, the timestamp is added after the base file name and before the extension, if present;
  * Example: `myDb.db` => `myDb_20231003-1439.db`
* For in-memory databases, the timestamp is added after the base name.
  * Example: `myDb` => `myDb_20231003-1439`

## On the server

The relevant piece of [configuration](configuration-file/) is:

{% code lineNumbers="true" %}
```yaml
backup:
  backupDir: backups/
  numFiles: 3
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

#### `backupDir`

_Line 2, string, mandatory_

The directory to save the backups to. It must exist, and cannot be the same as the database file.

{% hint style="danger" %}
It is very much recommended to "reserve" the directory for just the backups.
{% endhint %}

#### `numFiles`

_Line 3, number, mandatory_

After each backup, `sqliterg` checks if there are more than a specified number of files (configured here) in the backup directory. If so, deletes the older ones until the desired number is reached.

{% hint style="danger" %}
That's why it's _really_ recommended to "reserve" the directory. Any file can be deleted, not just past backups.
{% endhint %}

#### `execution` policy

_Lines 4-11, object_

This object sets the means of execution of a backup. Within it are:

#### >`onCreate`

_Line 5, boolean_

If `true`, the backup will be executed at database creation, i.e. when

* a database file is configured, but does not exist at startup;
* an in-memory database is configured.

It's not very useful, as the backup will be empty. It's there for uniformity with [macros](macros.md#greater-than-oncreate).

If an error occurs in the backup of a file-based database, the new file is deleted.

#### >`onStartup`

_Line 6, boolean_

If `true`, the backup will be executed at database startup, i.e. when `sqliterg` starts.

If an error occurs in the backup of a file-based database, and the file was not present when the application was started up, the new file is deleted.

{% hint style="info" %}
If a backup is configured for both `onCreate` and `onStartup` and the database is new, the backup will be performed only once.
{% endhint %}

#### >`period`

_Line 7, number_

The backup can be configured to be run periodically. If this configuration is present and > 0, the backup will be executed every _n_ minutes, where _n_ is the period configured.

The first execution will be after _n_ minutes from the startup, then after _2n_ and so on.

#### >`webService`

_Lines 8-11_

Last, but again not least, it's possible to call a web service to execute a backup.

It's callable via POST or GET, at the following URL:

{% code lineNumbers="true" %}
```url
http://<host>:<port>/<db_name>/backup
```
{% endcode %}

The endpoint can be authenticated with a token-based setup. See the [relevant documentation](the-web-services/authentication.md#token-based-auth-for-macros-and-backup) for more info.
