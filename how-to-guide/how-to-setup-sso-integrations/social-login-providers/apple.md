# Connect Apps to Apple

## Prerequisite

To configure "Sign in with Apple" for Authgear, you will need to fulfil the following:

1. Register an Apple Developer Account. Apple Enterprise Account does not support "Sign in with Apple"
2. Register your own domain.
3. Your domain must be able to send and receive emails.
4. Set up [Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework)(SPF) for your domain.
5. Set up [DomainKeys Identified Mail](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail)(DKIM) for your domain.
6. Create an "App ID" by adding a new "Identifier" [here](https://developer.apple.com/account/resources/identifiers/list), choose app IDs, enable "Sign in with Apple" enabled.
7. Create a "Services ID" by adding a new "Identifier" [here](https://developer.apple.com/account/resources/identifiers/list), choose service IDs, enable "Sign in with Apple".\

8.  Click "Configure" the Next to "Sign in with Apple". In "Primary App ID" field, select app ID created above.

    <figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption><p>Select here to see Services IDs</p></figcaption></figure>
9. Fill in and verify the domain created above, add `https://<YOUR_AUTHGEAR_ENDPOINT>/sso/oauth2/callback/apple` to **Return URLs**
10. Create a "Key" following [this guide](https://developer.apple.com/help/account/manage-keys/create-a-private-key) with "Sign in with Apple" enabled. Click "Configure" next to "Sign in with Apple" and select "Primary App ID" with app ID created above. Keep the private key safe, you need to provide this later.

{% hint style="info" %}
Redirect URI has the form of `/sso/oauth2/callback/:alias`. The `alias` is used as the identifier of OAuth provider. You can configure the `alias` in Authgear Portal.
{% endhint %}

## Configure Sign in with Apple in Authgear Portal

1. In the portal, go to **Authentication > Social / Enterprise Login**.
2. Enable **Sign in with Apple**.
3. Fill in the **Client ID** with the **Service ID** obtained above.
4. In **Apple Developer Portal**, view key information of the "Key" created above.
5. Jot down the **Key ID** and download the key text file (`.p8` file).
6. Copy the content in the key text file to **Client Secret** text area in **Authgear Portal.**.
7. Fill in **Key ID** field using the **Key ID** obtained from step 5.
8. In **Apple Developer Portal**, click username on the top right corner, click **View Membership**.
9. Find the **Team ID** from **Membership Information**, fill in **Team ID** field in Authgear portal.
10. **Save** the settings.

🎉Done! You have just added Sign in with Apple to your apps!
