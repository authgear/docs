# Connect Websites to WeChat

{% hint style="info" %}
**WeChat Open Platform account (微信开放平台账号)** is different from **WeChat Official account (微信公众平台账号)**. Authgear supports integrating WeChat Login with a WeChat Open Platform account.
{% endhint %}

## Prerequisite

* Register a [WeChat Open Platform account (微信开放平台账号)](https://open.weixin.qq.com/).
* Register a Web Application (网站应用).

See [Appendix: Create web app on WeChat Open Platform](wechat-web.md#appendix-create-web-app-on-wechat-open-platform) for more details.

## Get the information from WeChat Open Platform

Once your mobile app is approved on the platform, you will see the word ":white\_check\_mark: <mark style="color:green;">已通过</mark>" in the app details page.

* Get the `appid` (**Client ID**)

<figure><img src="../../../.gitbook/assets/wechat-web-appid.png" alt="where to find appid"><figcaption><p>where to find appid</p></figcaption></figure>

* Get the `appsecret` (**Client Secret**). It will only be shown once. You need to re-generate if you lose it.

<figure><img src="../../../.gitbook/assets/wechat-web-appsecret.png" alt="where to find appid"><figcaption><p>where to find appid</p></figcaption></figure>

* Get the `原始ID` (**Account ID**) of your WeChat Open Platform account.

<figure><img src="../../../.gitbook/assets/wechat-open-platform-account-id.png" alt="where to find account ID"><figcaption><p>where to find account ID</p></figcaption></figure>

## Configure Sign in with WeChat in the Authgear portal

1. Sign in to the Authgear portal.
2. Select your project.
3. In the navigation menu, go to **Authentication > Social / Enterprise Login**.
4. Click **Add Connection**.
5. Select **WeChat Web / 网站应用**.
6. Fill in **Client ID** with the `appid`.
7. Fill in **Client Secret** with the `appsecret`.
8. Fill in **Account ID** with the `原始ID`.
9. Save.

## Done!

No further changes are needed. The Sign in with WeChat button should be shown in the signup / login page now.

## Appendix: Create web app on WeChat Open Platform

After logging into the Open Platform, go to the "网站应用" (Web App) and click "创建网站应用" to create a new web app

The following information are needed:

| Field      | Meaning in English                            | Usage                                                                                  |
| ---------- | --------------------------------------------- | -------------------------------------------------------------------------------------- |
| 网站应用名称     | Web app name                                  | The app name shown to the end-user when they login                                     |
| 英文名称       | Web app name in English                       | The app name shown to an English end-user when they login                              |
| 网站应用简介     | Web app description                           | A description for the approver to understand the app                                   |
| 英文简介       | Mobile app description in English             | Optional field                                                                         |
| 应用官网       | Official website of the web app               | The webpage should show the description of the app                                     |
| 网站应用图片     | App Icon                                      | A 28x28 and a 108x108 icon                                                             |
| 网站信息登记表扫描件 | Scanned copy of the Website Registration Form | A signed registration form about the web app and the business nature and company info. |
| 授权回调域      | Authorized redirect domain                    | Domain of your Authgear endpoint for example: `*.authgear.cloud`                       |
