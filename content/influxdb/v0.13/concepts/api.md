---
title: API Endpoints & Ports
menu:
  influxdb_013:
    weight: 80
    parent: concepts
---

## API reference

The InfluxDB API provides a simple way interact with the database.
It uses HTTP response codes, HTTP authentication, and responses are returned in
JSON.

The following sections assume your InfluxDB instance is running on `localhost`
port `8086` and HTTPS is not enabled.
Those settings [are configurable](/influxdb/v0.13/administration/config/).

## Endpoints

| Endpoint    | Description |
| :---------- | :---------- |
| [/ping](#ping) | Use `/ping` to check the status of your InfluxDB instance and your version of InfluxDB. |
| [/query](#query) | Use `/query` to query data and manage databases, retention policies, and users. |
| [/write](#write) | Use `/write` to write data to a pre-existing database. |

### /ping

The ping endpoint accepts both `GET` and `HEAD` HTTP requests.
Use this endpoint to check the status of your InfluxDB instance and your version
of InfluxDB.

#### Definition
```
GET http://localhost:8086/query
```
```
HEAD http://localhost:8086/query
```

#### Example

Extract the version of your InfluxDB instance in the `X-Influxdb-Version` field
of the header:
```bash
$ curl -sl -I localhost:8086/ping

HTTP/1.1 204 No Content
Request-Id: 7d641f0b-e23b-11e5-8005-000000000000
X-Influxdb-Version: 0.13.x
Date: Fri, 04 Mar 2016 19:01:23 GMT
```

#### Status Codes and Responses

The response body is empty.

| HTTP Status Code   | Description    |
| :----------------- | :------------- |
| 204      | Success! Your InfluxDB instance is up and running.      |

### /query

The `/query` endpoint accepts `GET` and `POST` HTTP requests.
Use this endpoint to query data and manage databases, retention policies,
and users.

#### Definition

```
GET http://localhost:8086/query
```
```
POST http://localhost:8086/query
```

#### Verb usage

| Verb  | Query Type |
| :---- | :--------- |
| GET   | Use for all queries that start with: <br><br> `SELECT`* <br><br> `SHOW`   |
| POST  | Use for all queries that start with: <br><br> `ALTER` <br><br> `CREATE` <br><br> `DELETE` <br><br> `DROP` <br><br> `GRANT` <br><br> `KILL` <br><br> `REVOKE` |

\* The only exception is `SELECT` queries that include an [`INTO` clause](/influxdb/v0.13/query_language/data_exploration/#the-into-clause).
Those `SELECT` queries require a `POST` request.

#### Query String Parameters

| Query String Parameter | Optional/Required | Definition |
| :--------------------- | :---------------- |:---------- |
| chunk_size=\<number_of_points> | Optional | Set the number of points returned per batch. By default, InfluxDB returns points in batches of 10,000 points. |
| db=\<database_name> | Required for database-dependent queries (most `SELECT` queries and `SHOW` queries require this parameter). | Sets the target [database](/influxdb/v0.13/concepts/glossary/#database) for the query. |
| epoch=[h,m,s,ms,u,ns] | Optional | Returns epoch timestamps with the specified precision. By default, InfluxDB returns timestamps in RFC3339 format with nanosecond precision. |
| p=\<password> | Optional if you haven't [enabled authentication](/influxdb/v0.13/administration/authentication_and_authorization/#set-up-authentication). Required if you've enabled authentication. | Sets the password for authentication if you've enabled authentication. Use with the query string parameter `u`. |
| pretty=true | Optional | Enables pretty-printed JSON output. While this is useful for debugging it is not recommended for production use as it consumes unnecessary network bandwidth. |
| rp=\<retention_policy_name> | Optional | Sets the target [retention policy](/influxdb/v0.13/concepts/glossary/#retention-policy-rp) for the query. InfluxDB queries the `DEFAULT` retention policy if you do not specify a retention policy.  |
| u=\<username> | Optional if you haven't [enabled authentication](/influxdb/v0.13/administration/authentication_and_authorization/#set-up-authentication). Required if you've enabled authentication. | Sets the username for authentication if you've enabled authentication. The user must have read access to the database. Use with the query string parameter `p`. |

#### Request Body

```
--data-urlencode "q=<InfluxQL query>"
```

Use curl's `--data-urlencode` encoding method for all queries.
Queries must be in [InfluxQL](influxdb/v0.13/query_language/), InfluxDB's query
language.

Delimit multiple queries with a semicolon.

#### Examples

Query data with a `SELECT` statement:
```
$ curl -GET 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas"'

{"results":[{"series":[{"name":"mymeas","columns":["time","myfield","mytag1","mytag2"],"values":[["2016-05-20T21:30:00Z",12,"1",null],["2016-05-20T21:30:20Z",11,"2",null],["2016-05-20T21:30:40Z",18,null,"1"],["2016-05-20T21:31:00Z",19,null,"3"]]}]}]}
```

Query data with a `SELECT` statement and an `INTO` clause:
```
$ curl -POST 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * INTO "newmeas" FROM "mymeas"'

{"results":[{"series":[{"name":"result","columns":["time","written"],"values":[["1970-01-01T00:00:00Z",4]]}]}]}
```

Query data with a `SELECT` statement and return pretty-printed JSON:
```
$ curl -GET 'http://localhost:8086/query?db=mydb&pretty=true' --data-urlencode 'q=SELECT * FROM "mymeas"'

{
    "results": [
        {
            "series": [
                {
                    "name": "mymeas",
                    "columns": [
                        "time",
                        "myfield",
                        "mytag1",
                        "mytag2"
                    ],
                    "values": [
                        [
                            "2016-05-20T21:30:00Z",
                            12,
                            "1",
                            null
                        ],
                        [
                            "2016-05-20T21:30:20Z",
                            11,
                            "2",
                            null
                        ],
                        [
                            "2016-05-20T21:30:40Z",
                            18,
                            null,
                            "1"
                        ],
                        [
                            "2016-05-20T21:31:00Z",
                            19,
                            null,
                            "3"
                        ]
                    ]
                }
            ]
        }
    ]
}
```

Query data with a `SELECT` statement and return second precision epoch
timestamps:
```
$ curl -GET 'http://localhost:8086/query?db=mydb&epoch=s' --data-urlencode 'q=SELECT * FROM "mymeas"'

{"results":[{"series":[{"name":"mymeas","columns":["time","myfield","mytag1","mytag2"],"values":[[1463779800,12,"1",null],[1463779820,11,"2",null],[1463779840,18,null,"1"],[1463779860,19,null,"3"]]}]}]}
```

Create a database:
```
$ curl -POST 'http://localhost:8086/query' --data-urlencode 'q=CREATE DATABASE mydb'

{"results":[{}]}
```

Create a database using HTTP authentication:
```
$ curl -POST 'http://localhost:8086/query?u=myusername&p=mypassword' --data-urlencode 'q=CREATE DATABASE mydb'

{"results":[{}]}
```

Send multiple queries:
```
$ curl -GET 'http://localhost:8086/query?db=mydb&epoch=s' --data-urlencode 'q=SELECT * FROM "mymeas";SELECT mean("myfield") FROM "mymeas"'

{"results":[{"series":[{"name":"mymeas","columns":["time","myfield","mytag1","mytag2"],"values":[[1463779800,12,"1",null],[1463779820,11,"2",null],[1463779840,18,null,"1"],[1463779860,19,null,"3"]]}]},{"series":[{"name":"mymeas","columns":["time","mean"],"values":[[0,15]]}]}]}
```

#### Status codes and responses

Responses are returned in JSON.
Enable pretty-print JSON by including the query string parameter `pretty=true`.

##### Summary Table
<br>

| HTTP status code | Description |
| :--------------- | :---------- |
| 200 OK | Success! The returned JSON offers further information. |
| 400 Bad Request | Unacceptable request. Can occur with a syntactically incorrect query. The returned JSON offers further information. |

##### Examples
<br>
A successful request that returns data:
```
$ curl -i -GET 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "mymeas"'

HTTP/1.1 200 OK
[...]
{"results":[{"series":[{"name":"mymeas","columns":["time","myfield","mytag1","mytag2"],"values":[["2016-05-20T21:30:00Z",12,"1",null],["2016-05-20T21:30:20Z",11,"2",null],["2016-05-20T21:30:40Z",18,null,"1"],["2016-05-20T21:31:00Z",19,null,"3"]]}]}]}
```

A successful request that returns an error:
```
$ curl -i -GET 'http://localhost:8086/query?db=mydb1' --data-urlencode 'q=SELECT * FROM "mymeas"'

HTTP/1.1 200 OK
[...]
{"results":[{"error":"database not found: mydb1"}]}
```

An incorrectly formatted query:
```
$ curl -i -GET 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT *'

HTTP/1.1 400 Bad Request
[...]
{"error":"error parsing query: found EOF, expected FROM at line 1, char 9"}
```

### /write

The `/write` endpoint accepts `POST` HTTP requests.
Use this endpoint to write data to a pre-existing database.

#### Definition

```
POST http://localhost:8086/write
```

#### Query String Parameters

| Query String Parameter | Optional/Required | Description |
| :--------------------- | :---------------- | :---------- |
| db=\<database> | Required | Sets the target [database](/influxdb/v0.13/concepts/glossary/#database) for the write. |
| p=\<password> | Optional if you haven't [enabled authentication](/influxdb/v0.13/administration/authentication_and_authorization/#set-up-authentication). Required if you've enabled authentication. | Sets the password for authentication if you've enabled authentication. Use with the query string parameter `u`. |
| precision=[n,u,ms,s,m,h] | Optional | Sets the precision for the supplied Unix time values. InfluxDB assumes that timestamps are in nanoseconds if you do not specify `precision`. |
| rp=\<retention_policy_name> | Optional | Sets the target [retention policy](/influxdb/v0.13/concepts/glossary/#retention-policy-rp) for the write. InfluxDB writes to the `DEFAULT` retention policy if you do not specify a retention policy. |
| u=\<username> | Optional if you haven't [enabled authentication](/influxdb/v0.13/administration/authentication_and_authorization/#set-up-authentication). Required if you've enabled authentication. | Sets the username for authentication if you've enabled authentication. The user must have write access to the database. Use with the query string parameter `p`. |


#### Request Body

```
--data-binary '<Data in Line Protocol format>'
```

Use curl's `--data-binary` encoding method for all writes.
Using any encoding method other than `--data-binary` will likely lead to issues;
`d`, `--data-urlencode`, and `--data-ascii` may all strip out newlines or
introduce new, unintended formatting.
Data must be in the
[Line Protocol](/influxdb/v0.13/concepts/glossary/#line-protocol) format.

Options:

* Write several points to the database with one request by separating each point
by a new line.
* Write points from a file with the `@` flag.
The file should contain a batch of points in the Line Protocol format.
Individual points must be on their own line and separated by newline characters
(`\n`).
Files containing carriage returns will cause parser errors.

    We recommend writing points in batches of 5,000 to 10,000 points.
Smaller batches, and more HTTP requests, will result in sub-optimal performance.

#### Examples

Write a point to the database `mydb` with a nanosecond timestamp:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb" --data-binary 'mymeas,mytag=1 myfield=90 1463683075000000000'
```

Write a point to the database `mydb` with the local server's nanosecond timestamp:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb" --data-binary 'mymeas,mytag=1 myfield=90'
```

Write a point to the database `mydb` with a timestamp in seconds:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb&precision=s" --data-binary 'mymeas,mytag=1 myfield=90 1463683075'
```

Write a point to the database `mydb` and the retention policy `myrp`:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb&rp=myrp" --data-binary 'mymeas,mytag=1 myfield=90'
```

Write a point to the database `mydb` using HTTP authentication:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb&u=myusername&p=mypassword" --data-binary 'mymeas,mytag=1 myfield=91'
```

Write several points to the database `mydb` by separating points with a new line:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb" --data-binary 'mymeas,mytag=3 myfield=89
mymeas,mytag=2 myfield=34 1463689152000000000'
```

Write several points to the database `mydb` from the file `data.txt`:
```
$ curl -i -POST "http://localhost:8086/write?db=mydb" --data-binary @data.txt
```

A sample of the data in `data.txt`:
```
mymeas,mytag1=1 value=21 1463689680000000000
mymeas,mytag1=1 value=34 1463689690000000000
mymeas,mytag2=8 value=78 1463689700000000000
mymeas,mytag3=9 value=89 1463689710000000000
```

#### Status codes and responses

In general, status codes of the form `2xx` indicate success, `4xx` indicate
that InfluxDB could not understand the request, and `5xx` indicate that the
system is overloaded or significantly impaired.
Errors are returned in JSON.

##### Summary Table
<br>

| HTTP status code | Description    |
| :--------------- | :------------- |
| 204 No Content   | Success!      |
| 400 Bad Request  | Unacceptable request. Can occur with a Line Protocol syntax error or if a user attempts to write values to a field that previously accepted a different value type. The returned JSON offers further information. |
| 404 Not Found    | Unacceptable request. Can occur a user attempts to write to a database that does not exist. The returned JSON offers further information. |
| 500 Internal Server Error  | The system is overloaded or significantly impaired. Can occur if a user attempts to write to a retention policy that does not exist. The returned JSON offers further information. |

##### Examples
<br>
A successful write:
```
HTTP/1.1 204 No Content
```

Write a point with an incorrect timestamp:
```
HTTP/1.1 400 Bad Request
[...]
{"error":"unable to parse 'mymeas,mytag=1 myfield=91 abc123': bad timestamp"}
```

Write an integer to a field that previously accepted a float:
```
HTTP/1.1 400 Bad Request
[...]
{"error":"field type conflict: input field \"myfield\" on measurement \"mymeas\" is type int64, already exists as type float"}
```

Write a point to a database that doesn't exist:
```
HTTP/1.1 404 Not Found
[...]
{"error":"database not found: \"mydb1\""}
```

Write a point to a retention policy that doesn't exist:
```
HTTP/1.1 500 Internal Server Error
[...]
{"error":"retention policy not found: myrp"}
```
