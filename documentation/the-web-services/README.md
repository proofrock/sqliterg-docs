# 🇺🇳 The Web Services

When `sqliterg` is run, it creates a number of web services, to be exact 3 per each database that is configured.

These are:

## WS for Query Execution

{% code lineNumbers="true" %}
```url
http://<host>:<port>/<db_name>
```
{% endcode %}

**Documentation**: [Request](the-query-web-service/requests.md), [Response](the-query-web-service/responses.md), [Authentication](authentication.md#web-service-for-queries-main)

## WS for Macros (configurable)

{% code lineNumbers="true" %}
```url
http://<host>:<port>/<db_name>/macro/<macro_id>
```
{% endcode %}

**Documentation**: [Request](../macros.md#greater-than-webservice), [Authentication](authentication.md#token-based-auth-for-macros-and-backup)

## WS for Backup  (configurable)

{% code lineNumbers="true" %}
```url
http://<host>:<port>/<db_name>/backup
```
{% endcode %}

**Documentation**: [Request](../backup.md#greater-than-webservice), [Authentication](authentication.md#token-based-auth-for-macros-and-backup)

