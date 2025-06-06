---
description: Learn how to update a User profile's custom attributes on sign-up using Hooks
---

# Update user profile on sign-up using Hooks

Using [Hooks](../../customization/events-hooks/denohooks.md) you can put **extra information** into the user profile's [custom attributes](broken-reference) programmatically. This is useful for [Profile Enrichment](https://www.authgear.com/post/how-profile-enrichment-can-boost-your-product) where making your current customer data better by adding more details from outside sources.

Here are easy steps to achieve this:

**Step 1.** Make sure that you have an Authgear account. If you don't have one, you can [create it for free](https://accounts.portal.authgear.com/signup) on the Authgear website. Start by logging into your [Authgear dashboard](https://portal.authgear.com/).

**Step 2.** Go to **User Profile** → **Custom Attributes** page.

**Step 3.** Add 3 new attributes there, namely _city_, _name_, and _timezone_:

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64c3b36a761d2d11e1c5ea89_Untitled%20(5).png" alt=""><figcaption></figcaption></figure>

**Step 4**. Navigate to your Authgear Dashboard's **Advanced**->**Hooks** section.

**Step 5. Add** a new **Blocking Event**.

**Step 6.** Choose the Block Hook **Type** as the _TypeScript_ and set the Event option to _User_ _pre-create_. You will write a new Typescript function from scratch.

**Step 7.** Click on **Edit Script** under the **Config** option.

**Step 8.** Write a function logic for how you integrate any external API to populate custom attributes into the editor. For example.

```typescript
		
export default async function(e: EventUserPreCreate): Promise
			 {
  // API Key for IP Geolocation
  const apiKey = 'MY_API_KEY';
  // Any random IP address
	const ipAddress = '8.8.8.8' 

  // Fetch data from the IP Geolocation API
  const response = await fetch(`https://api.ipgeolocation.io/ipgeo?apiKey=${apiKey}&ip=${ipAddress}`);
  const data = await response.json();

return {
    is_allowed: true,
    mutations:{
      user: {
          custom_attributes: {
            "city": data.city, 
            "country": data.country_name,
            "timezone": data.time_zone.name
        }
      }
    },
  };
}
```

**Step 9.** Now if you navigate to **User Management** and **Add** a new user.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64c3b456e2d961a288183a98_Untitled%20(6).png" alt=""><figcaption></figcaption></figure>

**Step 10.** After the user is created, you should able to see custom attributes values have been updated for the user:

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64c3b46e2482cbed9e3a37af_Untitled%20(7).png" alt=""><figcaption></figcaption></figure>

#### Next steps

Once you learned how to update user profiles, now you can discover different ways of [accessing user profiles](broken-reference) in Authgear.
