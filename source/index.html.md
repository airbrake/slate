---
title: Airbrake API Reference

toc_footers:
  - <a href="https://airbrake.io">Airbrake.io</a>

search: true
---

# Authentication

> To authorize, use this code:

```shell
curl "api_endpoint_here?key=(PROJECT_KEY|USER_KEY)"
```

Airbrake uses API keys to restrict access to the API. There are several kinds of keys:

- Project API key (`PROJECT_KEY`) that is used to submit errors and track deploys. This key is what you configure the notifier agent in your app to use.
- User API key (`USER_KEY`) is used to access to the project data through Airbrake APIs. Each Airbrake user has their own key.
- User token (`USER_TOKEN`) that is identical to `USER_KEY`, but is valid for limited time.

Airbrake expects the API key to be included in all API requests to our servers in a query string that looks like the following:

`?key=(PROJECT_KEY|USER_KEY|USER_TOKEN)`

<aside class="notice">
You must replace `(PROJECT_KEY|USER_KEY|USER_TOKEN)` with your personal key.
</aside>

## Create user token v4

```shell
curl -d "email=EMAIL&password=PASSWORD" "https://airbrake.io/api/v4/sessions"
```

```json
{
  "token": "B20koMUuNDO0sep9rIzqomiQHkp4z7YpiN0P2Jmo0p9gElQsJ1z3qQYM23hTtVYY="
}
```

### HTTP request

`POST https://airbrake.io/api/v4/sessions`

### POST data

The API expects URL-encoded data.

Key | Example
--- | -------
email | User email, e.g. john@airbrake.com.
Password | User password, e.g. qwerty.

### Response

The API returns `200 OK` status code on success and JSON data.

Field | Comment
----- | -------
token | User token that can be passed to the API instead of `USER_KEY`.

# Pagination

Almost all list APIs support pagination if you need access to all items. By default only first 20 items are returned.

```json
{
  "collectionName": [item1, item2, ..., item20],
  "count": 12345
}
```

### HTTP request

Get first page:

`GET https://airbrake.io/api/v4/collectionName`

Get second page:

`GET https://airbrake.io/api/v4/collectionName?page=2`

Ask for 100 items per page:

`GET https://airbrake.io/api/v4/collectionName?limit=100`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | Page number.
limit | 20 | Number of items per page.

### Response

Field | Example | Comment
----- | ------- | -----------
collectionName | `projects` | Each API has different collection name. Some APIs return multiple collections.
count | 12345 | Total number of available items.
page | 2 | Actual page number that backend used internally.

# Cursor pagination

Some list APIs use cursor-based pagination, that only allows to fetch next and previous page.

```json
{
  "collectionName": [item1, item2, ..., item20],
  "start": "START_CURSOR",
  "end": "END_CURSOR"
}
```

### HTTP request

Get next page:

`GET https://airbrake.io/api/v4/collectionName?start=END_CURSOR`

Get previous page:

`GET https://airbrake.io/api/v4/collectionName?end=START_CURSOR`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
start |  | Starting position within a result set from which to retrieve results. Used to retrieve next page.
end |  | Ending position within a result set from which to retrieve results. Used to retrieve previous page.
limit | 20 | Number of items per page.

### Response

Field | Example | Comment
----- | ------- | -----------
collectionName | `projects` | Each API has different collection name. Some APIs return multiple collections.
start | abcdefg | Position of the first element in the result set.
end | abcdefg | Position of the last element in the result set.

# Performance Monitoring

## Route performance endpoint

With this endpoint you can report performance data for the routes in your
application, the data will then become available in your project's [Performance
Monitoring dashboard](https://airbrake.io/docs/performance-monitoring/performance-dashboard-features/).

**PUT data**

The API expects JSON data to be sent with this request. Your `PROJECT_ID` and
`PROJECT_KEY` are required to authenticate and report performance data.

> **PUT /api/v5/projects/PROJECT_ID/routes-stats**

```shell
curl -X PUT -H "Content-Type: application/json" \
"https://api.airbrake.io/api/v5/projects/PROJECT_ID/routes-stats?key=PROJECT_KEY" \
-d '{
  "environment": "production",
  "routes": [
    {
      "method": "GET",
      "route": "/drinks/:drink_id",
      "statusCode": 200,
      "time": "2019-09-19T18:00:00+00:00",
      "count": 1,
      "sum": 50.0,
      "sumsq": 2500.0,
      "tdigest": "AAAAAkA0AAAAAAAAAAAAAUdqYAAB"
    }
  ]
}'
```

Field | Required | type |Description
------|----------|----|--------
environment | false | String |This is the environment the route stat was reported from
routes[] | true | Array | An array of route objects describing their performance
routes/{i}/route | true | String | The path describing your route e.g.  `'/drinks/:drink_id'`
routes/{i}/method | true | String |The HTTP method as a string: `'GET'`, `'POST'`, `'PUT'`, ...
routes/{i}/statusCode | true | Integer | The response code returned: `201`, `301`, `404`, `500`, ...
routes/{i}/count | true | Integer | The number of requests to this route
routes/{i}/time | true | String | The UTC time of the route activity to the minute, in [RFC3339 format](https://tools.ietf.org/html/rfc3339) `'2019-09-19T18:00:00+00:00'`
routes/{i}/sum | true | Float | Your route's response time in miliseconds
routes/{i}/sumsq | true | Float | The sum above squared
routes/{i}/tdigest | true | String | This value holds the routes percentile info as a t-digest, [More info on t-digests](https://github.com/tdunning/t-digest)

### Responses

A successful `PUT` to this endpoint returns a `204` No Content.  If any
required fields are missing the `PUT` fails and returns a `400` Bad Request and
the message in the response will state which key is missing.

# Error notification v3

## Create notice v3

Notifies Airbrake that a new error has occurred in your application.

### POST data

The API expects JSON data.

See [POST Data Fields](#post-data-fields-v3) &
[POST Data Schema](#post-data-schema-v3).

### HTTP request

`POST https://airbrake.io/api/v3/projects/PROJECT_ID/notices?key=PROJECT_KEY`

```shell
curl -X POST -H "Content-Type: application/json" -d JSON "https://airbrake.io/api/v3/projects/PROJECT_ID/notices?key=PROJECT_KEY"
```

> Example JSON for the above request:

```json
{
  "errors": [
    {
      "type": "error1",
      "message": "message1",
      "backtrace": [
        {
          "file": "backtrace file",
          "line": 10,
          "function": "backtrace function",
          "code": {
            "1": "code",
            "2": "more code"
          }
        }
      ]
    },
    {
      "type": "error2",
      "message": "message2",
      "backtrace": [
        {
          "file": "backtrace file",
          "line": 10,
          "function": "backtrace function"
        }
      ]
    }
  ],
  "context": {
    "notifier": {
      "name": "notifier name",
      "version": "notifier version",
      "url": "notifier url"
    },
    "os": "Linux 3.5.0-21-generic #32-Ubuntu SMP Tue Dec 11 18:51:59 UTC 2012 x86_64",
    "hostname": "production-rails-server-1",
    "language": "Ruby 2.1.1",
    "environment": "production",
    "severity": "error",

    "version": "1.1.1",
    "url:": "http://some-site.com/example",
    "rootDirectory": "/home/app-root-directory",

    "user": {
      "id": "12345",
      "name": "root",
      "email": "root@root.com"
    }
  },
  "environment": {
    "PORT": "443",
    "CODE_NAME": "gorilla"
  },
  "session": {
    "basketId": "123",
    "userId": "456"
  },
  "params": {
    "page": "3",
    "sort": "name",
    "direction": "asc"
  }
}
```

### POST data fields v3

Field | Required | Description
------|----------|------------
errors | true | An array of objects describing the error that occurred.
errors/{i}/type | false | The class name or type of error that occurred.
errors/{i}/message | false | A short message describing the error that occurred.
errors/{i}/backtrace | false | An array of objects describing each line of the error's backtrace.
errors/{i}/backtrace/{i}/file | false | The full path of the file in this entry of the backtrace.
errors/{i}/backtrace/{i}/line | false | The file's line number in this entry of the backtrace.
errors/{i}/backtrace/{i}/column | false | The line's column number in this entry of the backtrace.
errors/{i}/backtrace/{i}/function | false | When available, the function or method name in this entry of the backtrace.
errors/{i}/backtrace/{i}/code | false | Current line of code plus a few lines around.
context | false | An object describing additional context for this error.
context/notifier | false | An object describing the notifier client library.
context/notifier/name | false | The name of the notifier client submitting the request, e.g. "airbrake-js".
context/notifier/version | false | The version number of the notifier client submitting the request, e.g. "1.2.3".
context/notifier/url | false | A URL at which more information can be obtained concerning the notifier client.
context/environment | false | The name of the server environment in which the error occurred, e.g. "staging", "production", etc.
context/severity | false | How severe the error that occurred. Allowed values: `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, `emergency`, `invalid`.
context/component | false | The component or module in which the error occurred. In MVC frameworks like Rails, this should be set to the controller. Otherwise, this can be set to a route or other request category.
context/action | false | The action in which the error occurred. If each request is routed to a controller action, this should be set here. Otherwise, this can be set to a method or other request subcategory.
context/os | false | Details of the operating system on which the error occurred.
context/hostname | false | The hostname of the server on which the error occurred.
context/language | false | Describe the language on which the error occurred, e.g. "Ruby 2.1.1".
context/version | false | Describe the application version, e.g. "v1.2.3".
context/url | false | The application's URL.
context/userAgent | false | The requesting browser's full user-agent string.
context/userAddr | false | The IP address of the user that triggered the notice.
context/remoteAddr | false | The IP address of the server that reported the notice.
context/rootDirectory | false | The application's root directory path.
context/user/id | false | If applicable, the current user's ID.
context/user/name | false | If applicable, the current user's username.
context/user/email | false | If applicable, the current user's email address.
environment | false | An object containing the current environment variables. Where the key is the variable name, e.g. `{ "PORT": "443", "CODE_NAME": "gorilla" }`.
session | false | An object containing the current session variables. Where the key is the variable name, e.g. `{ "basket_total": "1234", "user_id": "123" }`.
params | false | An object containing the request parameters. Where the key is the parameter name, e.g. `{ "page": "3", "sort": "desc" }`.

### POST data schema v3

The JSON POST data schema for the v3 notifier API.

```json
{
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "errors": {
      "type": "array",
      "required": true,
      "items": {
        "type": "object",
        "additionalProperties": false,
        "properties": {
          "type": {"type": "string", "required": false},
          "message": {"type": "string", "required": false},
          "backtrace": {
            "type": "array",
            "required": false,
            "items": {
              "type": "object",
              "additionalProperties": false,
              "properties": {
                "file": {"type": "string", "required": false},
                "function": {"type": "string", "required": false},
                "line": {"type": "number", "required": false},
                "column": {"type": "number", "required": false},
                "code": {"type": "object", "required": false}
              }
            }
          }
        }
      }
    },
    "context": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "environment": {"type": "string"},
        "component": {"type": "string"},
        "action": {"type": "string"},
        "os": {"type": "string"},
        "language": {"type": "string"},
        "environment": {"type": "string"},
        "severity": {"type": "string"},
        "version": {"type": "string"},
        "url": {"type": "string"},
        "userAgent": {"type": "string"},
        "userAddr": {"type": "string"},
        "remoteAddr": {"type": "string"},
        "rootDirectory": {"type": "string"},
        "hostname": {"type": "string"},
        "notifier": {
          "type": "object",
          "required": false,
          "additionalProperties": false,
          "properties": {
            "name": {"type": "string", "required": false},
            "version": {"type": "string", "required": false},
            "url": {"type": "string", "required": false}
          }
        },
        "user": {
          "type": "object",
          "required": false,
          "additionalProperties": false,
          "properties": {
            "id": {"type": "string"},
            "username": {"type": "string"},
            "name": {"type": "string"},
            "email": {"type": "string"}
          }
        }
      }
    },
    "environment": {"type": "object"},
    "session": {"type": "object"},
    "params": {"type": "object"}
  }
}
```

### Response

On success, the API returns a `201 Created` status with the following JSON data.

Field | Comment
----- | -------
id | The UUID of the newly created error notice. This can be used to [query the status of this error notice](#show-notice-status-v4).
url | A URL that will take you to the error on the Airbrake dashboard.

**Note**: a success response means that the data has been received and accepted
for processing. Use the `url` or `id` in the response to query the status of an
error. This will tell you if the error has been processed, or if it has been
rejected for reasons including invalid JSON and rate limiting.

If errors are sent in excess of the account's rate limit, the API will return a
`420 Too many requests in last minute - account is rate limited` status.

If the request body size exceeds **64KB**, the API will reject the notice and
return a `413 Request Entity Too Large` status.

# Projects v4

## List projects v4

```shell
curl "https://airbrake.io/api/v4/projects?key=USER_KEY"
```

```json
{
  "projects": [
    {
      "id": 1,
      "name": "Airbrake project name",
      "deployId": "1",
      "deployAt": "2014-09-26T17:37:33.638348Z",
      "noticeTotalCount": 1,
      "rejectionCount": 1,
      "fileCount": 1,
      "deployCount": 1,
      "groupResolvedCount": 1,
      "groupUnresolvedCount": 1
    }
  ]
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects?key=USER_KEY`

### Response

The API returns `200 OK` status code on success.

## Show project v4

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID?key=USER_KEY"
```

```json
{
  "project": {
    "id": 1,
    "name": "Airbrake project name",
    "deployId": "1",
    "deployAt": "2014-09-26T17:37:33.638348Z",
    "noticeTotalCount": 1,
    "rejectionCount": 1,
    "fileCount": 1,
    "deployCount": 1,
    "groupResolvedCount": 1,
    "groupUnresolvedCount": 1
  }
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID?key=USER_KEY`

### Response

The API returns `200 OK` status code on success.

# Deploys v4

## Create deploy v4

```shell
curl -X POST -H "Content-Type: application/json" -d '{"environment":"production","username":"john","email":"john@smith.com","repository":"https://github.com/airbrake/airbrake","revision":"38748467ea579e7ae64f7815452307c9d05e05c5","version":"v2.0"}' "https://airbrake.io/api/v4/projects/PROJECT_ID/deploys?key=PROJECT_KEY"
```

### HTTP request

`POST https://airbrake.io/api/v4/projects/PROJECT_ID/deploys?key=PROJECT_KEY`

### POST data

The API expects JSON data.

Key | Example
--- | -------
environment | production
username | john
email | john@smith.com
repository | https://github.com/airbrake/airbrake
revision | 38748467ea579e7ae64f7815452307c9d05e05c5
version | v2.0

### Response

The API returns `201 Created` status code on success.

## List deploys v4

The API returns list of project deploys. See [Pagination](#pagination) section for supported query parameters and response fields.

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/deploys?key=USER_KEY"
```

```json
{
  "deploys": [
    {
      "environment": "production",
      "username": "john",
      "email": "john@smith.com",
      "gravatarId": "767fc9c115a1b989744c755db47feb60",
      "repository": "https://github.com/airbrake/airbrake",
      "revision": "38748467ea579e7ae64f7815452307c9d05e05c5",
      "version": "v2.0"
    }
  ],
  "count": 1
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/deploys?key=USER_KEY`

### Response

The API returns `200 OK` status code on success.

## Show deploy v4

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/deploys/DEPLOY_ID?key=USER_KEY"
```

```json
{
  "deploy": {
    "environment": "production",
    "username": "john",
    "email": "john@smith.com",
    "repository": "https://github.com/airbrake/airbrake",
    "revision": "38748467ea579e7ae64f7815452307c9d05e05c5",
    "version": "v2.0"
  }
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/deploys/DEPLOY_ID?key=USER_KEY`

### Response

The API returns `200 OK` status code.

# Groups v4

> **Note:**
> Groups are also known as "error groups" in the web app UI.

## List groups v4

The API returns list of groups. See [Pagination](#pagination) section for supported query parameters and response fields.

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/groups?key=USER_KEY"
```

```json
{
  "groups": [
    {
      "id": 1,
      "projectId": 1,
      "resolved": false,
      "errors": [
        {
          "type": "error type",
          "message": "error message",
          "backtrace": [
            {
              "file": "/path/to/file",
              "function": "func_name",
              "line": 1,
              "column": 0
            }
          ]
        }
      ],
      "context": {
        "environment": "production"
      },
      "lastDeployId": "1",
      "lastDeployAt": "2014-09-26T17:37:33.638348Z",
      "lastNoticeId": "1",
      "lastNoticeAt": "2014-09-26T17:37:33.638348Z",
      "noticeCount": 1,
      "noticeTotalCount": 1,
      "createdAt": "2014-09-26T17:37:33.638348Z"
    }
  ],
  "count": 1
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/groups?key=USER_KEY`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
deploy_id | | Filters groups by deploy id.
archived | | When set to `true` returns archived groups.
muted | | When set to `true` returns muted groups.
start_time | | Returns groups created after `start_time`.
end_time | | Returns groups created before `end_time`.
order | last_notice | Sorts groups by `last_notice`, `notice_count`, `weight` and `created` fields.

### Response

The API returns `200 OK` status code on success.

## Show group v4

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID?key=USER_KEY"
```

```json
{
  "group": {
    "id": 1,
    "projectId": 1,
    "resolved": false,
    "errors": [
      {
        "type": "error type",
        "message": "error message",
        "backtrace": [
          {
            "file": "/path/to/file",
            "function": "func_name",
            "line": 1,
            "column": 0
          }
        ]
      }
    ],
    "context": {
      "environment": "production"
    },
    "lastDeployId": "1",
    "lastDeployAt": "2014-09-26T17:37:33.638348Z",
    "lastNoticeId": "1",
    "lastNoticeAt": "2014-09-26T17:37:33.638348Z",
    "noticeCount": 1,
    "noticeTotalCount": 1,
    "createdAt": "2014-09-26T17:37:33.638348Z"
  }
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID?key=USER_KEY`

### Response

The API returns `200 OK` status code on success.

## Mute group v4

This API removes group from the default list and disables all notifications.

```shell
curl -X PUT "https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/muted?key=USER_KEY"
```

### HTTP request

`PUT https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/muted?key=USER_KEY`

### Response

The API returns `204 No Content` status code on success.

## Unmute group v4

Opposite of the mute group.

```shell
curl -X PUT "https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/unmuted?key=USER_KEY"
```

### HTTP request

`PUT https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/unmuted?key=USER_KEY`

### Response

The API returns `204 No Content` status code on success.

## Delete group v4

The API permanently deletes group.

<aside class="warning">This operation can not be undone. Use it with care.</aside>

```shell
curl -X DELETE "https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID?key=USER_KEY"
```

### HTTP request

`DELETE https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID?key=USER_KEY`

## List groups across all projects v4

The API returns list of groups across all projects. See [Cursor pagination](#cursor-pagination) section for supported query parameters and response fields.

```shell
curl "https://airbrake.io/api/v4/groups?key=USER_KEY"
```

```json
{
  "groups": [
    {
      "id": 1,
      "projectId": 1,
      "resolved": false,
      "errors": [
        {
          "type": "error type",
          "message": "error message",
          "backtrace": [
            {
              "file": "/path/to/file",
              "function": "func_name",
              "line": 1,
              "column": 0
            }
          ]
        }
      ],
      "context": {
        "environment": "production"
      },
      "lastDeployId": "1",
      "lastDeployAt": "2014-09-26T17:37:33.638348Z",
      "lastNoticeId": "1",
      "lastNoticeAt": "2014-09-26T17:37:33.638348Z",
      "noticeCount": 1,
      "noticeTotalCount": 1,
      "createdAt": "2014-09-26T17:37:33.638348Z"
    }
  ],
  "end": "d312cff95ca275d7d4"
}
```

### HTTP request

`GET https://airbrake.io/api/v4/groups?key=USER_KEY`

### Response

The API returns `200 OK` status code on success.

## Show group statistics

The API returns statistics for the group.

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/stats?key=USER_KEY"
```

```json
{
  "projectId": PROJECT_ID,
  "groupId": GROUP_ID,
  "accepted": [904, 2013],
  "limited": [6784, 4245],
  "time": ["2018-06-12T09:00:00Z", "2018-06-12T10:00:00Z"]
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/stats?key=USER_KEY`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
period |  | Aggregates results for the period. Common periods are: 1m, 5m, 15m, 30m, 2h, 3h, 6h, 12h, and 24h.
time__gte | | Filters results by `time >= VALUE`.
limit | 100 | Limits number of results.

### Response

The API returns `200 OK` status code on success.

# Notices v4

> **Note:**
> Notices are also known as "error occurrences" in the web app UI.

## List notices v4

The API returns list of group notices. See [Pagination](#pagination) section for supported query parameters and response fields.

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/notices?key=USER_KEY"
```

```json
{
  "notices": [
    {
      "id": "1272072008639432675",
      "projectId": 1,
      "groupId": "1272072008639432676",
      "createdAt": "2014-10-22T03:00:00.407Z"
    }
  ],
  "count": 1
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/groups/GROUP_ID/notices?key=USER_KEY`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
version | | Filters notices by version, e.g. `version=1.0`.

### Response

The API returns `200 OK` status code on success.


### Response

The API returns `204 NO CONTENT` status code on success.

## Show notice status v4

The API returns notice status:

- `processed` - notice is processed. `groupId` contains notice group id.
- `rejected` - notice is rejected. `message` contains the reason, e.g. "app version is 1.2.1, wanted >= 1.3".
- `archived` - notice is archived according to the project retention limit.
- `not_found` - notice does not exist or is being processed by Airbrake.


### HTTP request

Note that `NOTICE_UUID` is returned by [error notification API v3](#create-notice-v3).

`curl "https://airbrake.io/api/v4/projects/PROJECT_ID/notice-status/NOTICE_UUID?key=USER_KEY"`


```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/notice-status/NOTICE_UUID?key=USER_KEY"
```

```json
{
  "code": "processed",
  "groupId": "1"
}
```

### Response

The API returns `200 OK` status code on success and JSON data.

Field | Comment
----- | -------
code | `processed`, `rejected`, `archived` or `not_found`.
groupId | `groupId` contains notice group id if notice is processed.

# Project activities v4

The API returns list of project activities. See [Pagination](#pagination) section for supported query parameters and response fields.

## List project activities v4

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/activities?key=USER_KEY"
```

```json
{
  "activities": [
    {
      "projectId": 1,
      "userId": 1,
      "userName": "Mr. Smith",
      "userGravatarId": "8b7d4e7f9fddecc8d93d73b2c01c0549",
      "activity": "group.resolved",
      "trackableId": 1,
      "trackableType": "Group",
      "createdAt": "2014-07-22T11:59:01.791773+03:00",
      "updatedAt": "2014-07-22T11:59:01.791773+03:00"
    },
    {
      "projectId": 1,
      "userId": 1,
      "userName": "Mr. Smith",
      "userGravatarId": "8b7d4e7f9fddecc8d93d73b2c01c0549",
      "activity": "group.resolved",
      "trackableId": 2,
      "trackableType": "Group",
      "createdAt": "2014-07-22T12:59:01.791773+03:00",
      "updatedAt": "2014-07-22T12:59:01.791773+03:00"
    }
  ],
  "count": 42
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/activities?key=USER_KEY`

### Response

The API returns `200 OK` status code on success.

## Show project statistics

The API returns statistics for the project.

```shell
curl "https://airbrake.io/api/v4/projects/PROJECT_ID/stats?key=USER_KEY"
```

```json
{
  "projectId": PROJECT_ID,
  "accepted": [904, 2013],
  "limited": [0, 0],
  "overQuota": [0, 0],
  "time": ["2018-06-12T09:00:00Z", "2018-06-12T10:00:00Z"]
}
```

### HTTP request

`GET https://airbrake.io/api/v4/projects/PROJECT_ID/stats?key=USER_KEY`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
period |  | Aggregates results for the period. Common periods are: 1m, 5m, 15m, 30m, 2h, 3h, 6h, 12h, and 24h.p
time__gte | | Filters results by `time >= VALUE`.
limit | 100 | Limits number of results.

### Response

The API returns `200 OK` status code on success.

# iOS crash reports v3

## Create iOS crash report v3

```shell
curl -X POST -H "Content-Type: application/json" -d '{"report":"REPORT_TEXT"}' "https://airbrake.io/api/v3/projects/PROJECT_ID/ios-reports?key=PROJECT_KEY"
```

### HTTP request

`POST https://airbrake.io/api/v3/projects/PROJECT_ID/ios-reports?key=PROJECT_KEY`

### POST data

The API expects JSON data.

Key | Example
--- | -------
report |
context | Same as in create notice API.

### Response

The API returns `201 Created` status code on success.
