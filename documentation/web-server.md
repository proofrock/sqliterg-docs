# 🌐 Embedded Web Server

`sqliterg` embeds a web server for static resources. This may be useful if you want to call it from a web page, to serve it on the same port as `sqliterg` itself, thus avoiding the need for CORS configurations.

To activate it, specify the `--serve-dir` [commandline switch](running.md#serve-dir), with the directory to serve.&#x20;

The contents of that directory will be served under the "`/`" virtual path (i.e. the root), with the GET method. Of course, the specified directory must exist.

{% hint style="info" %}
The connections to the database(s) are served as POST, so there's no overlap.
{% endhint %}

The `--index-file` [commandline switch](running.md#index-file) redirects to a specific index file when serving static assets and no file is specified in the URL.&#x20;

If not specified, `index.html` will be used.

{% hint style="warning" %}
The Web Server is purposely limited in scope: while it's plenty fast, it doesn't implement custom headers, HTTPS and many other features that could be useful. `sqliterg` is not, and doesn't want to be, a web server; if the feature is too limited, it's much better to use a [dedicated server](../integrations/reverse-proxy.md) for the static parts.
{% endhint %}
