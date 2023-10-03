# 🔓 Authentication

The authentication schema is different for the "main" web service (credentials-based) and for the macro and backup web services (token-based).

## Web Service for Queries ("main")

### On the server

The `auth`node represents the structure in the YAML companion file that instructs `sqliterg` to protect that db with authentication.

Going back to these snippets of [the configuration file](../configuration-file/):

{% code lineNumbers="true" %}
```yaml
[...]
auth:
  authErrorCode: 499
  mode: HTTP_BASIC
  byCredentials:
    - user: myUser1
      password: myCoolPassword
    - user: myUser2
      hashedPassword: b133a0c0e9bee3be20163d...
 [...or...]
auth:
  mode: INLINE
  byQuery: SELECT 1 FROM AUTH WHERE USER = :user AND PASSWORD = :password
```
{% endcode %}

If a database is protected with auth and the client provides wrong credentials, or doesn't provide any, the HTTP answer will be `401 Unauthorised` or a custom code specified by you (see line 3).

The `auth` block is not mandatory. If provided, the database will be protected with it; if omitted, no authentication is requested. If you provide one, it will be ignored.

#### `mode`

_Lines 4, 12; string; mandatory_

The first parameter is `mode`, that indicates the method that the client is required to use to authenticate. It can be:

* `HTTP_BASIC`: the client needs to use [HTTP basic authentication](https://it.wikipedia.org/wiki/Basic\_access\_authentication);
* `INLINE`: the credentials needs to be specified in the JSON request. See [below](authentication.md#credentials-in-the-request-inline-mode).

#### `authErrorCode`

_Line 3; number_

If this parameter is not specified, an authentication error will return the standard `401 Not Authorized`.&#x20;

Often a browser will react to this by displaying a standard authentication dialog; if this is not desirable (because the auth has a custom implementation, for example) it may be needed to specify an alternative error code. The `authErrorCode` configuration allows to do exactly this.

#### Providing the credentials

_Lines 5-9, 13; object; mandatory_

You can see that there are two methods to configure the resolution of the credentials on the server.

#### `byCredentials`

Provide a set of credentials in the config file itself, as in Lines 6-9.\
\
You can specify the password as plain text (ensure that the file is not world-readable!) or as SHA-256 hashes. See [below](authentication.md#generating-hashes) to learn how to hash passwords.

{% hint style="info" %}
If both `password` and `hashedPassword` are provided, the former "wins".
{% endhint %}

#### `byQuery`

Provide a query that will be executed in the database, as in Line 13.\
\
The query SQL must contain two placeholders, `:user` and `:password`, that will be replaced by the server with the username and password provided by the client.\
\
If the query returns at least one result, the credentials are valid; if it returns 0 records, access will be denied.

#### Generating hashes

In order to generate hashes for the password, you can use an online service like [this](https://emn178.github.io/online-tools/sha256.html), but it's better not to trust anything online. In Linux or MacOS you can instead use this one-liner (thet doesn't save the key to bash history):

{% code lineNumbers="true" %}
```bash
read -p Key: -rs srg_token && \
  echo && \
  echo -n $srg_token | \
  shasum -a 256 - | \
  head -c 64 && \
  echo
```
{% endcode %}

This will read a string from the stdin without echoing it, and outputs the hash to use.

{% hint style="warning" %}
Be careful not to include any whitespace in the text to hash, including any carriage return.
{% endhint %}

### On the client

{% hint style="danger" %}
The password are passed in cleartext, so it is better to be on a protected connection like HTTPS (e.g. by using a reverse proxy). See the [security](../../security.md#authentication) page for further information.
{% endhint %}

#### Credentials in the request (`INLINE` mode)

When a database is protected with authentication in [`INLINE` mode](authentication.md#mode), the client needs to specify the credentials in the request itself. Simply include a node like this:

{% code lineNumbers="true" %}
```json
{
    "credentials": {
        "user": "myUser1",
        "password": "myCoolPassword"
    },
    [...]
```
{% endcode %}

If the token verification fails, the response will be returned after 1 second, to prevent brute forcing. The wait time is per database: different failed requests for the same database will "stack", while different databases will work concurrently.

## Token-based auth for macros and backup

When [macros](../macros.md) and [backup](../backup.md) subsystems are exposed as a web service, token based authentication must be set up.&#x20;

### On the server

Relevant configuration, for both services:

{% code lineNumbers="true" %}
```yaml
      webService:
        authErrorCode: 499
        authToken: ciao
        # or hashedAuthToken: b133a0c0e9bee3be20163d...
```
{% endcode %}

#### `authErrorCode`

_Line 2_

Same as [the one discussed before](authentication.md#autherrorcode).

#### `authToken`, `hashedAuthToken`

_Lines 3, 4_

The token value, can also be specified as a [hash](authentication.md#generating-hashes).

{% hint style="info" %}
If both `authToken` and `hashedAuthToken` are provided, the former "wins".
{% endhint %}

### On the client

When calling the URL, a `token` query parameter must be specified, like this:

{% code lineNumbers="true" %}
```url
http://<host>:<port>/<db_name>/macro/<macro_id>?token=ciao
```
{% endcode %}

The token is always plaintext, even if the token specified server-side is hashed.

If the token verification fails, the response will be returned after 1 second, to prevent brute forcing. The wait time is per database: different failed requests for the same database will "stack", while different databases will work concurrently.
