---
layout: post
title: How To Query Redash via HTTP
author: Xiang Ye
description:
permalink: /:title/
categories: [Database]
tags: [database, redash]
---

# How To Query Redash via HTTP #

Redash becomes popular now, but it doesn't support REST APIs well. This post demonstrates a simple way to query redash via http.

## Server Settings ##

On server side, we first need to get user's API key which is used by client to do query. And then we need to create a query (client cannot run SQL query directly, but it can invoke existing server query and get result).

### Get API Key ###

See following figure:

![Get API Key](/images/20200326-how-to-query-redash-via-http/get-api-key.png)

### Create a Query ###

Since client can only invoke existing query, we must create a query so that client can use.

Click menu `"Create->Query"` to create a new query, and you can always modify it by clicking `"Edit Source"` button on that query's page. See following figure:

![Create Query](/images/20200326-how-to-query-redash-via-http/create-query.png)

NOTE: SQL accepts parameters which is wrapped by `"{{}}"`, in this example, two parameters are being used: `"start"` and `"limit"`.

## Client Side ##

Redash provides very limited support for REST APIs. A query cannot be done easily in a single step. Instead, there are three steps:

- Refresh server query with new parameters.
- Poll server's job status until it finish (succeed or fail).
- If it succeeds, read result from server.

### Refresh Query ###

Here is **`"refresh"`** API's information:

- method:
    - POST
- path:
    - **{redash_url}**/api/query/**{query_id}**/refresh
- parameters:
    - api_key: user's api key
    - SQL parameters: the name should be in `"p_{parameter-name}"` format
- return:
    - json    

Here is an example:

```
redash_url = https://redash.example.com
query_id = 580
api_key = 4JNYxxxxxxxxxxxxxxxxxxxxxxxxxxxCL6
SQL parameter "start": p_start = 1584332708788
SQL parameter "limit": p_limit = 10

POST https://redash.example.com/api/queries/580/refresh?api_key=4JNYxxxxxxxxxxxxxxxxxxxxxxxxxxxCL6&p_start=1584332708788&p_limit=10
```

And here is the response:

```
{
    "job": {
        "status": 2,
        "error": "",
        "id": "f646a216-a879-4170-9332-e86a4ec7b3f3",
        "query_result_id": null,
        "updated_at": 0
    }
}
```

The `"id"` is `"job_id"` which is used in next step to poll job's status.

### Poll Job Status ###

Here is **`"jobs"`** API's information:

- method:
    - GET
- path:
    - **{redash_url}**/api/jobs/**{job_id}**
- parameters:
    - api_key: user's api key
- return:
    - json    

Here is an example:

```
redash_url = https://redash.example.com
job_id = f646a216-a879-4170-9332-e86a4ec7b3f3
api_key = 4JNYxxxxxxxxxxxxxxxxxxxxxxxxxxxCL6

GET https://redash.example.com/api/jobs/f646a216-a879-4170-9332-e86a4ec7b3f3?api_key=4JNYxxxxxxxxxxxxxxxxxxxxxxxxxxxCL6
```

And here is the response:

```
{
    "job": {
        "status": 3,
        "error": "",
        "id": "f646a216-a879-4170-9332-e86a4ec7b3f3",
        "query_result_id": 8831,
        "updated_at": 0
    }
}
```

**NOTE:**

Client should keep polling job's status until it becomes 3 or 4.

- **3**: succeeded, use `"query_result_id"` as `"result_id"` to get result in next step.
- **4**: failed.

### Get Result ###

Here is **`"results"`** API's information:

- method:
    - GET
- path:
    - **{redash_url}**/api/query/**{query_id}**/results/**{result_id}**.json
- parameters:
    - api_key: user's api key
- return:
    - json    

Here is an example:

```
redash_url = https://redash.example.com
query_id = 580
result_id = 8831
api_key = 4JNYxxxxxxxxxxxxxxxxxxxxxxxxxxxCL6

GET https://redash.example.com/api/query/580/results/8831.json?api_key=4JNYxxxxxxxxxxxxxxxxxxxxxxxxxxxCL6
```

And here is the response:

```
{
  "query_result": {
    "retrieved_at": "2020-03-26T18:26:30.369053+00:00",
    "query_hash": "e5e90d67bed55e094e94159a7bc6f0e6",
    "query": "SELECT *\nFROM default.xxxxxxxx_rt\nWHERE r_time > 1584332708788\nORDER BY r_time\nLIMIT 10",
    "runtime": 4.69629907608032,
    "data": {
      "rows": [
        {
          "r_time": 1584332709971,
          "file_hash": "7f5e07bc449380683103dd58cde4aee4361d8f8f"
        },
        {
          "r_time": 1584332710346,
          "file_hash": "04fa710b3d00278199c771ee80b5c5bf80ec9aad"
        },
        {
          "r_time": 1584332711207,
          "file_hash": "04fa710b3d00278199c771ee80b5c5bf80ec9aad"
        }
      ],
      "columns": [
        {
          "friendly_name": "r_time",
          "type": "integer",
          "name": "r_time"
        },
        {
          "friendly_name": "file_hash",
          "type": "string",
          "name": "file_hash"
        }
      ]
    },
    "id": 8831,
    "data_source_id": 10
  }
}
```
