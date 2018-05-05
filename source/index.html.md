---
title: Sqlify API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://sqlify.io/account'>Sign Up for an API Key</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Sqlify API reference!

With our API, you can do everything you do on our [website](/) and more from your own servers or applications.

# Authentication

For your API calls to go through, you need to authenticate using your key.

To find your key, head over to your [account page](/account).

Sqlify uses the header `X-Api-key` to read your api key.

`X-Api-Key: ${API_KEY}`

To make it easier to use with cURL, you can also use basic HTTP authentication to pass your key, like this (don't forget the colon, so curl doesn't prompt for a password):


`curl https://api.sqlify.io/v1/files -u "${API_KEY}:"`

<aside class="notice">
Remember to replace <code>${API_KEY}</code> with your own API key.
</aside>

# Response format

> Sucess response example

```json
{
  "status": "ok",
  "message": "file created",
  "data": {}
}
```

> Error response example

```json
{
  "status": "error",
  "message": "size required",
  "data": {}
}
```

All Sqlify API calls have the following response format:

Name | Type | Description
--------- | ------- | -----------
status | string | The result of the request, it can be `ok` in case of success or `error` in case of failure
message | string | The message result, if the status is `error` check this field for more information to see what went wrong
data | any | Returned data will always be in this field.

# Converting files

> Converting JSON to CSV specifying a JSON path schema:

```shell
echo '[{"id":1,"name":"Mark"}, {"id":2,"name":"Maria"}]' > example.json

curl https://api.sqlify.io/v1/pipelines/sqlify/convert/run \
  -u "${API_KEY}:" \
  -F "file=@example.json" \
  -F "options[to]=csv" \
  -F "options[schema][0][name]=id" \
  -F "options[schema][0][path]=\$[*].id" \
  -F "return_output=true"
```

> Will return:

```csv
"id"
"1"
"2"
```
> Converting JSON to CSV without a schema:

```shell
echo '[{"id":1,"name":"Mark"}, {"id":2,"name":"Maria"}]' > example.json

curl https://api.sqlify.io/v1/pipelines/sqlify/convert/run \
  -u "${API_KEY}:" \
  -F "file=@example.json" \
  -F "options[to]=csv" \
  -F "options[output][separator]=;" \
  -F "return_output=true"
```

> Will return:

```csv
"id";"name"
1;"Mark"
2;"Maria"
```

If you just want to convert a file as simply as possible, this is the best way to do it.

The convert endpoint is:

`POST https://api.sqlify.io/v1/pipelines/sqlify/convert/run`

And accepts these params:

Name | Type | Description
--------- | ------- | -----------
file | multi-part file | The file that you want to convert
file_id | string | The file id that you want to convert, you need to specify a file or a file_id
return_output | bool | Indicate wether you want the result file to be returned, or a [file object response](#get-a-file) instead.
options | object | options for conversion
options[from] | string | (optional) The input format that you want, can be `csv` or `json`, it will be autodetected if not specified
options[to] | string | The output format that you want, can be `csv`, `sql` or `json`
options[schema] | [field](#field-object)[] | A list of fields, this is where you specify what fields you want to convert and their types
options[output] | object | Encoding options for the output format
options[input] | object | Dencoding options for the input format

# Files

## File object

Name | Type | Description
--------- | ------- | -----------
file_id | string | Unique file id.
type | string | MIME type of the file.
size | integer | Size of the file in bytes.
schema | field[] | A list of fields defining the schema of the file.
runs | run[] | A list of runs that used this file as an input.
download_url | string | Url to get the raw data of this file.
download_url | string | Url to upload the chunks of the file.

## Field object

Name | Type | Description
--------- | ------- | -----------
name | string | Name of the field
type | string | Type of the field, needs to be one of `string`, `integer`, `float`, `json`
path | string | The source of the field value.

## Get all your files

```shell
curl https://api.sqlify.io/v1/files -u "${API_KEY}:"
```

> The above command returns JSON structured like this:

```json
{
  "status": "ok",
  "message": "",
  "data": [
    {
      "file_id": "966065f3-35c1-4f6d-b0f6-f2e960896973",
      "type": "application/json",
      "size": 1024,
      "schema": [],
      "runs": [],
      "download_url": "https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973/data"
      "upload_url": "https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973/data"
    }
  ]
}
```

This endpoint retrieves all the files that belong to your account.

### HTTP Request

`GET https://api.sqlify.io/v1/files`

### Result

Name | Type | Description
--------- | ------- | -----------
data | file[] | List of files

## Get a file

```shell
curl https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973 -u "${API_KEY}:"
```

> The above command returns JSON structured like this:

```json
{
  "status": "ok",
  "message": "",
  "data": [
    "file_id": "966065f3-35c1-4f6d-b0f6-f2e960896973",
    "type": "application/json",
    "size": 1024,
    "schema": [],
    "runs": [],
    "download_url": "https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973/data"
    "upload_url": "https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973/data"
  ]
}
```

Retrieve a single file by it's id.

### HTTP Request

`GET https://api.sqlify.io/v1/files/${FILE_ID}`

### Result

Name | Type | Description
--------- | ------- | -----------
data | file | File object


## Create a file

Theres a few ways you can create a file, via a URL, simple file upload or chunked file upload.

### Create a file via a URL

`POST https://api.sqlify.io/v1/files`

#### Params

Name | Type | Description
--------- | ------- | -----------
url | string | The URL where the API should get the data from, make sure it's publicly accessible


```shell
curl -X POST https://api.sqlify.io/v1/files \
    -u "${API_KEY}:" \
    -F "file=@data.json"
```

> The above command returns JSON structured like this:

```json
{
  "status": "ok",
  "message": "",
  "data": [
    "file_id": "966065f3-35c1-4f6d-b0f6-f2e960896973",
    "type": "application/json",
    "size": 1024,
    "schema": [],
    "runs": [],
    "download_url": "https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973/data"
    "upload_url": "https://api.sqlify.io/v1/files/966065f3-35c1-4f6d-b0f6-f2e960896973/data"
  ]
}
```

### Create a file via a simple file uplaod

We recommend using this method if you just want to upload a file and is less than 200Mb.

`POST https://api.sqlify.io/v1/files.`

#### Params

Name | Type | Description
--------- | ------- | -----------
file | file | A multi-form part representing your file

### Result

All the calls to create a file return the created file.

Name | Type | Description
--------- | ------- | -----------
data | file | File object


## Chunked upload for large files

We recommend using this method if your file is bigger than 200Mb.

Create the file it first, specifying the total size of the file:

### Create the file

`POST https://api.sqlify.io/v1/files`

Name | Type | Description
--------- | ------- | -----------
size | integer | The total size of your file.

Now all you need to do is split the file and upload the chunks to this endpoint.

### Upload a chunk

`POST https://api.sqlify.io/v1/files/${FILE_ID}/data`

Name | Type | Description
--------- | ------- | -----------
file | file | A muti-from part representing your chunk.
size | integer | The size of the chunk in bytes, all the chunks must be multiple of **5MiB (5 * 1024 * 1024 = 5242880)** but the last
offset | integer | The offset of the chunk in bytes.

### Return Code

The upload chunk endpoint will return `201 Created` when the last chunk is uploaded, `200 Sucess` and for the rest of chunks.

## Update a file

You can update a file to set it's schema.

`POST https://api.sqlify.io/v1/files/${FILE_ID}`

#### Params

Name | Type | Description
--------- | ------- | -----------
schema | field[] | List of fields
schema.name[] | string | Name of the field
schema.type[] | string | Type of the field, needs to be one of `string`, `integer`, `float`, `json`
schema.path[] | string | The source of the field value. In case of a CSV file, the path will be the column index, or a JSON path expression in case of a JSON File.

# Runs

A run is the object holding the information of the execution of a step. A step is a process that takes a file and outputs another. Pipelines are a list of steps.

## Run object

Name | Type | Description
--------- | ------- | -----------
id | string | Unique run id.
step | string | Name of the step that the run is executing.
status | string | The run status, it can be `queued`, `running`, `failed` or `completed`
finished | bool | Indicates if the run has finished, same as having a `failed` or `completed` status
output_file_id | string | File id of the result, it will only be populated if the run is `completed`
error_code | string | An error code in case the run has failed, for example: `limit_exceeded`
error_message | string | An error message describing the failure.
error_details | string | A more detailed error message.

## Get a run

```shell
curl https://api.sqlify.io/v1/runs/2f123527-86d4-439f-be17-b579450196f4 \
  -u "${API_KEY}:" 
```

```json
{
   "status":"ok",
   "message":"",
   "data": {
     "step":"sqlify/detect-schema",
     "status":"queued",
     "output_file_id":null,
     "id":"2f123527-86d4-439f-be17-b579450196f4",
     "finished":false
  }
}
```

Check the state of a run.

### HTTP Request

`GET https://api.sqlify.io/v1/runs/${RUN_ID}`

Name | Type | Description
--------- | ------- | -----------
data | run | Run object

## Queue runs

```shell
curl https://api.sqlify.io/v1/pipelines/sqlify/convert-to-csv/run \
  -u "${API_KEY}:" \
  -d "file_id=9c7c6d10-ed9d-4560-88db-9a80487fedbd"
```

```json
{
   "status":"ok",
   "message":"pipeline queued",
   "data":[
      {
         "step":"sqlify/detect-schema",
         "status":"queued",
         "output_file_id":null,
         "id":"2f123527-86d4-439f-be17-b579450196f4",
         "finished":false
      },
      {
         "step":"sqlify/export-csv",
         "status":"queued",
         "output_file_id":null,
         "id":"55f01194-3bfb-496d-947c-43ffb267171b",
         "finished":false
      }
   ]
}
```


A **step** is a process that runs on an input file and produces an output file.

The `sqlify/detect-schema` step analyzes the file and writes the schema in the file specified.

The `sqlify/export-*` steps take the data, cast it to the schema and encode it in the desired format.

A **pipeline** is a group of steps that get are chained together.

These are the **pipelines** that we have available:

Name |  Steps | Description
--------- | ------- | -----------
`sqlify/convert-to-csv` | `sqlify/detect-schema` `sqlify/export-csv` | Ouputs a CSV file
`sqlify/convert-to-json` | `sqlify/detect-schema` `sqlify/export-json` | Ouputs a JSON file
`sqlify/convert-to-sql` | `sqlify/detect-schema` `sqlify/export-sql` | Ouputs a SQL file

The detect schema step will skip files that already have a schema, unless the `rewrite_schema` option is passed. So you can specify the schema in your file and run the pipeline safely.

### HTTP Request

`GET https://api.sqlify.io/v1/pipelines/${PIPELINE}/run`

#### Params

Name | Type | Description
--------- | ------- | -----------
file_id | string | The input file id
options | object | Any options that you want to pass to the steps _(optional)_

#### Response

Name | Type | Description
--------- | ------- | -----------
data | run[] | List of runs
