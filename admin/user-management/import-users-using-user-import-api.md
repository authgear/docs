---
description: >-
  Use the user import API to bulk import users from external systems to your
  Authgear project
---

# Import Users using User Import API

Some ways to add users to your Authgear project include; using the **Add User** UI in Authgear Portal, using the **createUser()** mutation in Admin API, and last but not least, having the user accounts created using sign-up page on AuthUI.

A common downside of all the above-listed methods is that they do not support batch import of users. Meaning, that you have to add users one by one. This isn't ideal for importing multiple users from existing legacy systems to Authgear. For adding bulk users, there is the User Import API.

In this post, you'll learn what the User Import API is and see examples of how to import bulk users to an Authgear project.

## User Import API

The User Import API is an API that supports the bulk import of users from another system to an Authgear project. This API is not part of the Admin API GraphQL. However, a valid [Admin API JWT token](https://docs.authgear.com/reference/apis/admin-api/authentication-and-security) is required to access the endpoints of the User Import API.

### Endpoints

The following are the endpoints for the User Import API.

#### Initiate Import

Use this endpoint to start a new user import task.

```
POST - /_api/admin/users/import
HTTP/1.1
Host: <Your Authgear Project domain>
Authorization: Bearer <Admin API JWT Token>
Content-type: application/json
Body: {
    "identifier": "email",
    "records": [
        {
            "email": "user@example.com",
            "email_verified": true,
            "password": {
                "type": "bcrypt",
                "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
            }
        }
    ]
}

```

#### Check Status

Use this endpoint to verify the status of an existing user import task.

```
GET - /_api/admin/users/import/{ID}
HTTP/1.1
Host: <Your Authgear Project domain>
Authorization: Bearer <Admin API JWT Token>
```

{% hint style="info" %}
To learn more detailed information about the User Import API (such as endpoints, inputs, and responses), see the [User Import API Reference](../../api-reference/apis/user-import-api.md).
{% endhint %}

## Usage Example

In this section, you can find code for a simple example of using the User Import API to add multiple users to an Authgear project.

### Prerequisites

To follow this example and be able to run the code on your local machine, you must have the following:

* An Authgear account. Sign up for free [here](https://accounts.portal.authgear.com/signup).
* [Node.js](https://nodejs.org/) installation on your local computer.
* Install Express.js by running the following command from your project directory: `npm install express`.

### Step 1: Get Admin API JWT

As mentioned earlier in this post, the User Import API requires the Admin API JWT to access. In this step, we'll generate an Admin API JWT using Node.js.

First, install JsonWebToken (a Node package for generating JWT) by running the following command:

```sh
npm install jsonwebtoken
```

The following code shows how to get the token:

```javascript
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

See our post on [Admin API Authentication](https://docs.authgear.com/reference/apis/admin-api/authentication-and-security) for a more detailed guide (and examples for other programming languages/frameworks) on how to get your key ID, and private key and generate Admin API JWT.

### Step 2: Import Users from a JSON Document

To import users to your Authgear project using the User Import API, make a POST HTTP(S) request to the Initiate Import endpoint (`/_api/admin/users/import`).&#x20;

The user data you wish to import must be provided as a JSON input in the HTTP(S) request body as specified in the [API input format](../../api-reference/apis/user-import-api.md#input-format).

In the following steps, we'll use the node-fetch package to make HTTP requests to the User Import API. Hence, install node-fetch by running the following command:

```sh
npm install node-fetch
```

The following code sample demonstrates how to import 2 users from a JSON document that's stored in a simple constant (`const data`):

```javascript
const express = require("express");
const node_jwt = require('jsonwebtoken');
const fs = require('fs');
const fetch = require('node-fetch');
const app = express();
const port = 3002;

//TODO Place declaration of generateJWT() function here

app.get('/', (request, response) => {

    const jwt = generateJWT();
    const data = {
        "identifier": "email",
        "records": [
            {
                "email": "user1@example.com",
                "email_verified": true,
                "password": {
                    "type": "bcrypt",
                    "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
                }
            },
            {
                "email": "user2@example.com",
                "email_verified": false,
                "name": "John Doe",
                "given_name": "John",
                "family_name": "Doe",
                "password": {
                    "type": "bcrypt",
                    "password_hash": "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
                }
            }
        ]
    }

    const options = {
        method: 'POST',
        headers: { 'Content-type': 'application/json', 'Authorization': 'Bearer ' + jwt },
        body: JSON.stringify(data)
    };

    const appUrl = 'https://your-project.authgearapps.com'; // replace wuth your authgear project url

    fetch(`${appUrl}/_api/admin/users/import`, options)
        .then(result => result.json())
        .then(result => console.log(JSON.stringify(result)));

    response.send("Request sent using the follow JWT as Bearer: " + jwt + "See console for result");
});

app.listen(port, () => {
    console.log("server started! PORT: " + port);
});
```

If the user import was initiated successfully, you'll get a response that looks like this:

```json
{
  "id": "task_4WZ0V7EPT4GZ2ABVN03QXYZ122W835C1",
  "created_at": "2024-04-04T06:56:36.02508096Z",
  "status": "pending"
}
```

In the next step, we'll use the value of the `id` field from the above response to query the status of the import task.

### Step 3: Get the Status of the Import Task

In this step, we'll make a GET HTTP(S) request to the Check Status endpoint (`/_api/admin/users/import/{ID}`) to get the status of the user import task we initiated in the last step. You'll need to replace `{ID}` in the URL with the value of `id` in the previous response.

To do this, add a new route to the Express app that accepts the task `id` as a parameter and uses that `id` to query the status of the task. Here's the code for the route:

```javascript
app.get("/status/:id", (request, response) => {
    const jwt = generateJWT()

    const options = {
        method: 'GET',
        headers: { 'Content-type': 'application/json', 'Authorization': 'Bearer ' + jwt }
    };

    const appUrl = 'https://your-project.authgearapps.com';

    fetch(`${appUrl}/_api/admin/users/import/${request.params.id}`, options)
        .then(result => result.json())
        .then(result => console.log(JSON.stringify(result)));

    response.send("Request sent using the follow JWT as Bearer: " + jwt + "See console for result");
});
```

The response to the request to query the status of the import task will look like this:&#x20;

```json
{
  "id": "task_4WZ0V7EPT4GZ2ABVN03QXYZ122W835C1",
  "created_at": "2024-04-04T06:56:36.02508096Z",
  "status": "completed",
  "summary": {
    "total": 2,
    "inserted": 2,
    "updated": 0,
    "skipped": 0,
    "failed": 0
  },
  "details": [
    {
      "index": 0,
      "record": {
        "email": "user1@example.com",
        "email_verified": true,
        "password": {
          "password_hash": "REDACTED",
          "type": "bcrypt"
        }
      },
      "outcome": "inserted",
      "user_id": "0f0f65ee-4c7d-45a0-a740-bcbbfd3fcf06"
    },
    {
      "index": 1,
      "record": {
        "email": "user2@example.com",
        "email_verified": false,
        "family_name": "Doe",
        "given_name": "John",
        "name": "John Doe",
        "password": {
          "password_hash": "REDACTED",
          "type": "bcrypt"
        }
      },
      "outcome": "inserted",
      "user_id": "9c71fc29-6db6-4a18-aa73-774139fed16d",
      "warnings": [
        {
          "message": "email_verified = false has no effect in insert."
        }
      ]
    }
  ]
}
```

From the response, you can see the `status` of the entire task (import was `completed`), including a summary ( `{ "total": 2, "inserted": 2, "updated": 0, "skipped": 0, "failed": 0 }` ).

The `details` field contains an array of details such as the `outcome` for each user in the original JSON document.
