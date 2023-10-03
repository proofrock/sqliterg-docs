# 🚄 Performances

The target `make profile` executes performance tests, absolute and against the latest `ws4sqlite` binary.

{% hint style="info" %}
A recent Java JDK is necessary, and currently the scripts are for Linux and `x86_64`
{% endhint %}

The profiler submits 20'000 calls to the server, with each call covering pretty much all the various features.

See below for the call JSON. It's 8 calls, of which 2 `SELECT`s reading values and 6 modifying them.

This is a performance profile for a "bundled" build:

```
sqliterg v0.17.0. based on SQLite v3.41.2
...
Elapsed seconds in sqliterg: 46.325
```

&#x20;                                                         \=> **0.29ms** per call

This is for a "dynamic" build:

```
sqliterg v0.17.0. based on SQLite v3.41.2
...
Elapsed seconds in sqliterg: 25.999
```

&#x20;                                                         \=> **0.16ms** per call

The baseline `ws4sqlite` numbers are:

```
ws4sqlite v0.15.0, based on sqlite v3.41.2
...
Elapsed seconds in ws4sqlite: 52.702
```

Testing was made on a 2-core, 4-threads `i3-1115G4 @ 3.00GHz`.

## JSON of each call

{% code lineNumbers="true" %}
```json
{
    "credentials": {
        "user": "myUser",
        "password": "ciao"
    },
    "transaction": [
        {
            "statement": "DELETE FROM TBL"
        }, {
            "query": "SELECT * FROM TBL"
        }, {
            "statement": "INSERT INTO TBL (ID, VAL) VALUES (:id, :val)",
            "values": {
                "id": 0,
                "val": "zero"
            }
        }, {
            "statement": "INSERT INTO TBL (ID, VAL) VALUES (:id, :val)",
            "valuesBatch": [{
                    "id": 1,
                    "val": "uno"
                }, {
                    "id": 2,
                    "val": "due"
                }
            ]
        }, {
            "noFail": true,
            "statement": "INSERT INTO TBL (ID, VAL) VALUES (:id, :val, 1)",
            "valuesBatch": [{
                    "id": 1,
                    "val": "uno"
                }, {
                    "id": 2,
                    "val": "due"
                }
            ]
        }, {
            "statement": "INSERT INTO TBL (ID, VAL) VALUES (:id, :val)",
            "valuesBatch": [{
                    "id": 3,
                    "val": "tre"
                }
            ]
        }, {
            "query": "SELECT * FROM TBL WHERE ID=:id",
            "values": {
                "id": 1
            }
        }, {
            "statement": "DELETE FROM TBL"
        }
    ]
}

```
{% endcode %}
