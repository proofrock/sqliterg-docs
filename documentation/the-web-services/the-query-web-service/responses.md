# ❗ Responses

If you followed all of the chapters until now, in particular if you used[ the config file](../../configuration-file/) when starting ws4sql and submitted the [request](requests.md), you'll get the following response. Just as the request was "complete", this response illustrates all the necessary concepts.

{% code lineNumbers="true" %}
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
{% endcode %}

Everything ran successfully, but for node #4, whose failure was managed; and the transaction was committed.

If successful, the returned HTTP code is 200, and the response body consists of an array of results, one for each query in the request.

If unsuccessful, the HTTP code would have been 400 or 500, and the response body will report the error, as discussed [in the next chapter](errors.md).

Let's go through the nodes in `results`, one by one:

#### Result of a Query

_Nodes 1 and 2, lines 3-15_

These nodes represent the result of the execution of a `query`. The corresponding requests were:

{% code lineNumbers="true" %}
```json
{ "query": "SELECT * FROM TEMP" },
{
  "query": "SELECT * FROM TEMP WHERE ID = :id",
  "values": { "id": 1 }
},
```
{% endcode %}

A `query` is a statement that returns results in form of a result set.

In the results, you can see a node named `resultSet`, an array of objects. Each object represents a record returned by the query, and is a map (key-value) of all the fields in the record. The key is the field name, as returned by the database, and the value is the (typed) field value.

Notice that we selected for `*`, so the database auto-assigned the keys to the name of the fields.

#### Result of a (non-Query) Statement

_Node 3, lines 16-19_

This is the result of a `statement`. The request was:

```json
{
  "statement": "INSERT INTO TEMP (ID, VAL) VALUES (0, 'ZERO')"
},
```

A `statement` node is a SQL command that does _not_ return a result set; it usually alters the database, and it returns the number of rows changed by it.

Notice that DDL statements, like `CREATE` or `ALTER`, return 0 as the number of changed rows.

The `rowsUpdated` node contains this piece of information.

#### (Non-blocking) Error

_Node 4, lines 20-23_

The request was:

```json
{
  "noFail": true,
  "statement": "INSERT INTO TEMP (ID, VAL) VALUES (:id, :val)",
  "values": { "id": 1, "val": "a" }
},
```

This statement failed, but it was marked as `noFail`. So, it didn't rollback the whole transaction, but reported a failure for just this request, and in the end the transaction was committed.

See further discussion in the [relevant section](errors.md).

#### Result of a Batch Statement

_Node 5, lines 24-27_

The request was:

```json
{
  "statement": "^Q2",
  "valuesBatch": [
    { "id": 2, "val": "b" },
    { "id": 3, "val": "c" }
  ]
}
```

The statement was provided via a Stored Query, but this is just transparent, doesn't change the response in any way.

But this statement had two sets of values passed to it, a batch. We can see that the result contains this:

```json
"rowsUpdatedBatch": [ 1, 1 ]
```

So, the server returns a "rows updated" count for every item of the batch, aka for each iteration of the statement.
