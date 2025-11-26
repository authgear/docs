---
hidden: true
---

# Example: Only Allow Signups from Inside the Corporate Network using Hooks

Authgear Webhook is a feature that sends notifications in the form of HTTP requests to an external URL that you specify when certain events occur. For example, Webhook can send the user.pre\_create event just before a new user is created.

Webhooks also listen for response from your external services. For [blocking events](https://docs.authgear.com/how-to-guide/events-hooks#blocking-events), this response determines if the process that triggered the event should continue to completion or be terminated. In this post, we'll use a blocking event and a custom webhook endpoint to restrict user signup to only devices from inside a corporate network.

## How to Prevent or Allow Signups Using Webhooks

Configuring a webhook for the `user.pre-created` event will prevent Authgear from creating new users if your webhook endpoint returns `is_allowed: false` in its response. In contrast, a new user will be created successfully when the endpoint returns `is_allowed: true` in its response.

We will be using this behavior to prevent users from outside a corporate network from signing up by checking their IP address (which is provided in the payload of the webhook HTTP request from Authgear). When the IP address matches a specified allowed IP address, we set the value for `is_allowed` to `true`. Otherwise, we return `false`.

The following tutorial shows detailed steps for allowing signups from inside a corporate network only.

### Step 1: Configure Webhook on Authgear Portal

In this step, we'll create a webhook for the **User pre-create** event in the Authgear Portal. To do that, login to the Authgear Portal and navigate to **Advance** > **Hooks**. Next, under **Blocking Events** section, click on **Add**, then select **Webhook** as the type and **User pre-create** as the Event.

Enter the full URL that points to the page that will be receiving and processing the webhook requests in the Endpoint text field.

<figure><img src="../../.gitbook/assets/authgear-webhooks (1) (1).png" alt=""><figcaption></figcaption></figure>

Once you're done, click on the Save button at the top.

### Step 2: Create an Endpoint

You'll need to create an endpoint in your preferred programming language or framework and host it online. For this tutorial, we'll be creating a basic Express.js endpoint.

Create a new Express.js project and add the following code to `app.js`:

```javascript
const express = require("express");
const app = express();
const bodyParser = require('body-parser');
const port = process.env.PORT || 3002;
app.use(bodyParser.json());


app.post('/', (request, response) => {

    response.status(200).send(
        {
            "is_allowed": false,
            "reason": "some reason",
            "title": "some title"
          }
    );
});

app.listen(port, () => {
    console.log("server started!");
});
```

Let's walk through what the above code will do:

* Express creates an endpoint that accepts **POST** requests. (Note that the Authgear webhook only sends **POST** HTTP requests and your endpoint must support HTTPS).
* Deploying the above code and pointing your Authgear webhook endpoint to it should prevent any user from signing up.
* This is because the code returns `is_allowed: false` as a response to the Webhook. Hence terminating the signup process.
* You can also state a more descriptive reason for termination in the endpoint response using the `reason` field.

### Step 3: Read Data from Webhook Request

Authgear webhooks send data to the endpoint URL to enable developers to build custom features and more. In this step, we'll read the data from the **User pre-created** event call and get the IP address for the user trying to register.

To read the request body and output the value to the console add the following code to app.js on a new line just above `response.status(200)...`

```javascript
const requestPayload = request.body;
console.log(requestPayload)
```

Attempting to register a new user again should print the payload data similar to this to your endpoints console:

```javascript
{
    id: '0000000000022b55',
    seq: 142165,
    type: 'user.pre_create',
    payload: {
      user: {
        id: '413fe869-dba2-43a8-b671-95fd471b7950',
        created_at: '2023-08-14T03:44:36.383367Z',
        updated_at: '2023-08-14T03:44:36.473456Z',
        is_anonymous: false,
        is_verified: true,
        is_disabled: false,
        is_deactivated: false,
        is_anonymized: false,
        can_reauthenticate: true,
        standard_attributes: [Object],
        x_web3: [Object]
      },
      identities: [ [Object] ]
    },
    context: {
      timestamp: 1691984676,
      user_id: '413fe869-dba2-43a8-b671-95fd471b7950',
      triggered_by: 'user',
      audit_context: null,
      preferred_languages: [ 'en-US', 'en' ],
      language: 'en',
      ip_address: '123.456.78.009',
      user_agent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/116.0',
      app_id: 'my_test_app'
    }
  }
```

The data we're interested in for this example is `ip_address` in the `context` object (context.ip\_address). We'll use this IP address to determine whether a user is inside the corporate network.

Now, update app.js so that it returns `is_allowed: true` when the user's IP address is within the corporate network's IP address.

The code for app.js should look like this after the modification:

```javascript
app.post('/', (request, response) => {

    const requestPayload = request.body;
    console.log(requestPayload)

    const allowedIp = '123.456.78.009'; //replace the value with the corporate network IP address 
    if (requestPayload.context.ip_address == $orgIp) {

        //this user is from the organisation network, continue with sign up.
        response.status(200).type('json').send(
            {
                "is_allowed": true
              }
        );

    } else {
        //user has an IP address outside of the organisation network so disallow sign up.
        response.status(200).type('json').send(
            {
                "is_allowed": false,
                "reason": "You are not allow to sign up to this organisation",
                "title": "Sign-up not allowed!"
              }
        )
    }
    
});
```

You should replace the value for `allowedIp` with the IP address you wish to allow.

At this point, if you deploy your endpoint and try signing up from the allowed IP the process should complete without issue, and you've successfully implemented signups only from a corporate network.

## Only Allow Signups from Inside the Corporate Network using JavaScript/TypeScript Hooks

Authgear offers an alternative for regular webhooks called the [JavaScript/TypeScript hooks](https://docs.authgear.com/how-to-guide/events-hooks/denohooks). This kind of hook makes it possible to implement our earlier solution without hosting an external endpoint. In the following section, we'll walk through the steps for allowing signups from inside a corporate network only using the Typescript hook.

### Step 1: Configure TypeScript Hook on Authgear Portal

Navigate to **Advance** > **Hooks** in the Authgear portal. Next, click on **Add** under the **Blocking Events** section. Or, modify the previous webhook.

Select **TypeScript** as the type and **User pre-create** as the Event.

The third field in a TypeScript hook is different from a regular webhook. The field allows you to enter JavaScript code that the hook can execute when an event is triggered.

<figure><img src="../../.gitbook/assets/authgear-typescript-hook (1).png" alt=""><figcaption></figcaption></figure>

You can click on the **Edit Script** button near the Script field to open the hook script editor.

### Step 2: Write the JavaScript for the Hook

Modify the code in the script editor to the following:

```typescript
export default async function (e: EventUserPreCreate): Promise<HookResponse> {
  // Write your hook with the help of the type definition.
  const allowedIp = "123.456.78.009";
  if (e.context.ip_address == allowedIp) {
    return {
      is_allowed: true,
    };
  } else {
    return {
      is_allowed: false,
      reason: "You are not allow to sign up to this organisation",
      title: "Sign-up not allowed!",
    };
  }
}
```

Make sure to update the value for `allowedIp` to the IP address you wish to allow.

Once you're done, click on **Finish Editing** to return to the Hooks configuration page. Save your new hook by clicking on the **Save** button at the top of the configuration page.

### Step 3: Test the Hook

To test that your TypeScript hook is working properly, try signing up a new user from the IP address you specified earlier. Also, try signing up outside that IP address to see if everything works as expected.
