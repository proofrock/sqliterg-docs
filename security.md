# 🛡 Security

Here is a more detailed discussion of the various security features.

## Authentication

The first and most effective line of defense is certainly authentication, [documented here](documentation/the-web-services/authentication.md).

`sqliterg` allows a client to authenticate using a traditional credentials system. In other words, the client provides an username and a password.

The client can provide them in one of two ways:

* Using the standard HTTP Basic Authentication; or
* Inserting the credentials in the request JSON.

{% hint style="danger" %}
Beware that the password is sent in plaintext; if you are serving a database on an unprotected connection, it could be snooped. It's strongly recommended to use HTTPS through a reverse proxy.
{% endhint %}

On the server, the set of valid credentials can be configured in two ways:

* Defining a list of credentials in the configuration file (the password can be plaintext or hashed);
* Defining a SQL query that authenticates them, typically against a table in the database itself.

This mechanism is designed to be flexible and effective. Of course, a password is only safe if it's kept secret, so if you connect from a web app, it better not be hard-coded in the javascript. A better approach would be to ask for it to the user.

There is also the possibility to configure authentication for macros and backups, when called via an endpoint. For this, a `token` URL parameter can be supplied when calling the endpoint, and the authentication can be set in the YAML configuration for the db (plain or hashed).

By default, if auth fails a `401` error is returned. If this is not ideal (maybe because the reverse proxy intercepts it), the code can be configured in the YAML configuration for the db.

## Read-only databases

If you want to ensure that a database cannot be changed, even if access to it is somehow gained, it's possible to specify to open it as read only. Docs [here](documentation/configuration-file/#readonly).

## Stored Statements to prevent SQL Injection

If you call `sqliterg` from a web page, or even if you expose it on an unsecured channel, it's all too easy for someone to call it as if they were the web page, and execute random SQL on it.

In such a case, it may be useful to [define](documentation/configuration-file/stored-statements.md) a set of "safe" SQL statements, on the server side, and [restrict](documentation/configuration-file/#useonlystoredqueries) _every_ client to only use those.

For example, suppose you have a database with a list of personal information, keyed by social security number. If someone "ask" with a valid SSN, you want to give them the information, but of course you don't want to allow anyone to extract the whole list.

A solution is to define a query by SSN as the only executable statement; it's an effective measure. The complete discussion of such a case is [here](documentation/configuration-file/stored-statements.md#limiting-the-server-to-executing-stored-queries).

Also, to prevent SQL injection, be sure to use [parameters](documentation/the-web-services/the-query-web-service/requests.md#parameter-values-for-the-query-statement)!

## CORS Allowed Origin

If you access `sqliterg` from a web context, you'll probably need to configure CORS. The application lets you do it, and instead of `*` you can specify the calling address (to use as value for the Allow Origin header), so that only the calls from that address are allowed.

See the [docs](documentation/configuration-file/#corsorigin). Again, be sure to use HTTPS!

## Binding to a network interface

As most engines of the same kind, `sqliterg` allows to limit the server to a network interface, by binding to an address of the host. Docs [here](documentation/running.md#bind-host).

For example, this can be used to restrict access to the service only to requests coming from a VPN or another trusted network.

## Use a reverse proxy (if going on the internet)

All these measures are effective, but to complement them it's advisable to use HTTPS if you plan to expose `sqliterg` on the internet.

HTTPS configuration is _not_ part of `sqliterg`, because it would add a lot of code that is outside the scope of the application (KISS!). Instead, configure a reverse proxy to secure the HTTP connection - for example with a LetsEncrypt certificate.

Some options that are very easy to configure are [`Caddy`](https://caddyserver.com) or [LinuxServer's `Swag`](https://docs.linuxserver.io/general/swag) Docker image, which is based on `nginx`. Documentation [here](integrations/reverse-proxy.md).
