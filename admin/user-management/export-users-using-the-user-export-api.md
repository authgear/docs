---
description: Export users from your project into a CSV or JSON file
---

# Export Users using the User Export API

The Export User API offers a means for exporting user data such as user ID, email, phone number, etc from your Authgear project into a [CSV](https://datatracker.ietf.org/doc/html/rfc4180) or [ndjson](https://github.com/ndjson/ndjson-spec) file.

In this guide, you'll learn how to use the User Export API.

## User Export API

The User Export API allows developers to bulk export users into a file.&#x20;

The export process is asynchronous. That is, the process runs in the background. Hence, you will need to initiate an export task in one endpoint call and then, make an additional call to another endpoint to get the status of the export task.

To make HTTP(S) requests to Export User API endpoints, your application must be authenticated using an [Admin API JWT token](https://docs.authgear.com/reference/apis/admin-api/authentication-and-security) in the Bearer Authorization header. The API will return a "403 Forbidden" error if an invalid JWT is used.

The following are the two endpoints for the User Export API:

#### Initiate Export Task

<mark style="color:green;">`POST`</mark> `/_api/admin/users/export`

Use this endpoint to create a new user export task.

**Headers**

| Name          | Value                            |
| ------------- | -------------------------------- |
| Content-Type  | `application/json`               |
| Authorization | `Bearer <Admin API JWT Token>`   |
| Host          | `<Your Authgear Project domain>` |

**Body**

The Initiate Export endpoint accepts JSON input via an HTTP(S) request body. The following is an example of the input:

```json
{
  "format": "csv",
  "csv": {
    "fields": [
      {
        "pointer": "/sub",
        "field_name": "user_id"
      },
      {
        "pointer": "/email"
      }
    ]
  }
}
```

* The `format` field is where you specify the format of the export file. The value can be `csv` or `ndjson`.
* `csv`: use this field when `format` is set to `csv`. The value is an object with a `fields` property.
* `csv.fields`: you can use this field to list all the user attributes you want to include as fields in the CSV file. The value should be an array and each item in the array is an object with a `pointer` and an optional `field_name` property that describe a user attribute.&#x20;

#### Check Status

<mark style="color:green;">`GET`</mark> `/_api/admin/users/export/{Task ID}`

Use this endpoint to query the status of an existing export task. Replace `{Task ID}` with the task `id` returned in the response body of the initiate export endpoint.

**Headers**

| Name          | Value                            |
| ------------- | -------------------------------- |
| Authorization | `Bearer <Admin API JWT Token>`   |
| Host          | `<Your Authgear Project domain>` |

{% hint style="info" %}
See the [User Export API Reference](export-users-using-the-user-export-api.md#user-export-api) for more details about the endpoints, inputs, and pointers.
{% endhint %}

## Example: Using the User Export API

The following example shows how to use the User Export API in a Node.js application.

### Step 1: Get Admin API JWT

First, you need to get the Admin API JWT that will be used to authenticate requests to the endpoints.

To do that, install JsonWebToken (a Node package for generating JWT) by running the following command:

```sh
npm install jsonwebtoken
```

Now, create a `generateJWT()` function in your Express app to generate the JWT:

```javascript
const node_jwt = require('jsonwebtoken');
const fs = require("fs");

function generateJWT() {
    const project_id = ""; //Your authgear app id
    const key_id = ""; //you authgear key ID
    const expiresAt = Math.floor(Date.now() / 1000) + (60 * 60); //the current value means token will expire in 1 hour.
    
    //Payload to include in JWT
    const claims = {
        aud: project_id,
        iat: Math.floor(Date.now() / 1000) - 30,
        exp: expiresAt
    }
    const privateKey = fs.readFileSync("key.pem"); //Read value from the downloaded key file
    const header = { "typ": "JWT", "kid": key_id, "alg": "RS256" }
    const jwt = node_jwt.sign(claims, privateKey, { header: header });

    return jwt;
}
```

See [Admin API Authentication](https://docs.authgear.com/reference/apis/admin-api/authentication-and-security) for a more detailed guide on how to get your key ID, and private key and generate Admin API JWT using different programming languages.

### Step 2: Initiate User Export Task

Make an HTTP(S) POST request to the initiate export endpoint to initiate a new user export task.&#x20;

To do that, first, install the node-fetch package in your app using this command:

```sh
npm install node-fetch
```

Then add the following code to your application:

```javascript
const express = require("express");
const node_jwt = require("jsonwebtoken");
const fs = require("fs");
const fetch = require("node-fetch");
const app = express();
const port = 3002;

app.get("/export", (request, response) => {
  const jwt = generateJWT();
  const input = {
    format: "csv",
    csv: {
      fields: [
        {
          pointer: "/sub",
          field_name: "user_id",
        },
        {
          pointer: "/email",
        },
      ],
    },
  };

  const options = {
    method: "POST",
    headers: {
      "Content-type": "application/json",
      Authorization: "Bearer " + jwt,
    },
    body: JSON.stringify(input),
  };

  const appUrl = "https://your-project.authgearapps.com";

  fetch(`${appUrl}/_api/admin/users/export`, options)
    .then((result) => result.json())
    .then((result) => response.send(result))
    .catch((error) => response.send(error));
});

app.listen(port, () => {
  console.log("server started! PORT: " + port);
});
```

The response to the HTTP(S) request in this step should look like this:

**Response**

```json
{
    "result": {
        "id": "userexport_VWFAACAMB5V0BY1J7KK5NS3GV1TQAACQ",
        "created_at": "2024-10-07T21:06:24.361634372Z",
        "status": "pending",
        "request": {
            "format": "csv",
            "csv": {
                "fields": [
                    {
                        "pointer": "/sub",
                        "field_name": "user_id"
                    },
                    {
                        "pointer": "/email"
                    },
                    {
                        "pointer": "/mfa/emails"
                    }
                ]
            }
        }
    }
}
```

* `id`: the value of id is the task ID that can be used in the check status endpoint to query the task and get the download URL for the export file.

### Step 3: Check the Status of a User Export Task

In this step, we'll check the status of the export task we initiated by making an HTTP(S) request to the Check Status endpoint (`/_api/admin/users/export/{Task ID}`). Replace `{Task ID}` in the URL with the task ID for the export task in the previous task.

Add the following route to your app to check the status of an export task using the task ID:

```javascript
app.get("/status/:id", (request, response) => {
  const jwt = generateJWT();

  const options = {
    method: "GET",
    headers: {
      "Content-type": "application/json",
      Authorization: "Bearer " + jwt,
    },
  };

  const appUrl = "https://your-project.authgearapps.com";

  fetch(`${appUrl}/_api/admin/users/export/${request.params.id}`, options)
    .then((result) => result.json())
    .then((result) => response.send(result))
    .catch((error) => response.send(error));

  
});
```

When the status task is completed, the HTTP(S) response body will look like this:

**Response**

```json
{
    "result": {
        "id": "userexport_VWFAACAMB5V0BY1J7KK5NS3GV1TQAACQ",
        "created_at": "2024-10-07T21:06:24.361634372Z",
        "completed_at": "2024-10-07T21:06:24.551711713Z",
        "status": "completed",
        "request": {
            "format": "csv",
            "csv": {
                "fields": [
                    {
                        "pointer": "/sub",
                        "field_name": "user_id"
                    },
                    {
                        "pointer": "/email"
                    },
                    {
                        "pointer": "/mfa/emails"
                    }
                ]
            }
        },
        "download_url": "https://storage.googleapis.com/authgear-userexport-staging/example-userexport_userexport_VWFAACAMB5V0BY1J7KK5NS3GV1TQAACQ-20241007210624Z.csv?X-Goog-Algorithm=GOOG4-RSA-SHA256&X-Goog-Credential=authgear-server%40oursky-kube.iam.gserviceaccount.com%2F20241007%2Fauto%2Fstorage%2Fgoog4_request&X-Goog-Date=20241007T210641Z&X-Goog-Expires=59&X-Goog-Signature=abcd12e45&X-Goog-SignedHeaders=host"
    }
}
```

* `download_url` : open the URL in the value of `download_url` to download the exported users file.

**Note:** The result from a completed export task will expire after 24 hours. Hence, after 24 hours, you can no longer use the task ID associated with the task to generate a new download link.&#x20;
