---
description: Authentication for Laravel websites with Authgear and OAuth2
---

# Laravel

In this guide, you'll learn how to add user authentication to a Laravel app using Authgear as an OIDC provider.

Authgear supports multiple ways to allow users to log in to apps such as passwordless sign-in, phone OTP, and 2FA. In this post, we'll show you how to enable all these options in your Laravel app without worrying about the underlying logic.

This guide targets **Laravel 12** and **PHP 8.2+**.

### What You Will Learn

* How to create an Authgear Application.
* How to request an OAuth 2.0 authorization code from Authgear.
* How to get user info from Authgear using an OAuth 2.0 access token.
* How to replace Breeze's local password login with Authgear, linking users by their Authgear subject (`sub`).

### Prerequisites

To follow along with the example, you should have the following in place:

* A free Authgear account. [Sign up](https://accounts.portal.authgear.com/signup) if you don't have an account yet.
* PHP 8.2 or later, Composer 2, and Node.js 18 or later.

You can also clone the finished app from the [Laravel Example GitHub repo](https://github.com/authgear/authgear-example-laravel) and follow along.

#### What We Will Build

The example app we'll build uses Laravel Breeze for its UI scaffolding — the Blade layout, dashboard, and profile pages. Authgear is the only identity provider. We remove Breeze's local register, login, password-reset, and email-verification flows, so users sign in through Authgear and nothing else.

By handing authentication to Authgear, you get passwordless sign-in, phone OTP, 2FA, and more without writing or maintaining that logic yourself.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### How to Add User Authentication to Laravel with Authgear as an OAuth Provider

In this section, we'll walk through the complete steps for building the example app.

#### Step 1: Configure Authgear Application

Before you can use Authgear as an OAuth identity provider, set up an application on the Authgear portal.

Log in to Authgear and select a project. Navigate to the **Applications** section for your project. Create a new application or configure an existing one with **OIDC Client Application** as the Application Type, as shown below:

<figure><img src="../../.gitbook/assets/authgear-configure-project (1).png" alt=""><figcaption></figcaption></figure>

Click **Save** to go to the application configuration page. This page reveals the application credentials and OAuth 2.0 endpoints.

<figure><img src="../../.gitbook/assets/authgear-app-config-page (1).png" alt=""><figcaption></figcaption></figure>

Note down the Client ID, Client Secret, and the endpoints. You'll use them later in your Laravel project.

#### Step 2: Add a Redirect URI

While you're still on the application configuration page, scroll down to the URL section and click **Add URI**. Enter `localhost:8000/oauth/callback` in the text field if you'll run your Laravel app on your local machine. Click **Save**.

Authgear redirects users to this URI after authorization, so it must point to a valid route in your Laravel app.

#### Step 3: Create a Laravel Project

Create a new Laravel project by running the following command. This installs Laravel 12:

```sh
composer create-project laravel/laravel authgear-laravel-example
```

Open the project folder in your editor. Delete `resources/views/welcome.blade.php` and create a new `resources/views/index.blade.php` with this content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Authgear + Laravel Demo</title>
</head>
<body>
    <h1>Authgear + Laravel demo</h1>
    <p>This demo shows adding user authentication to a Laravel app with Authgear (OIDC / OAuth 2.0).</p>
    @if ($errors->any())
        <p style="color: #b00020;">{{ $errors->first() }}</p>
    @endif
    <p><a href="{{ route('login') }}">Login with Authgear</a></p>
</body>
</html>
```

Then point the root route at this view. Open `routes/web.php` and set:

```php
Route::get('/', function () {
    return view('index');
});
```

Run `php artisan serve` and open `localhost:8000` in a browser. You should see the landing page with a login link.

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

#### Step 4: Install Laravel Breeze

Breeze is the official starter kit for Laravel. Here we use it only for UI scaffolding — the Blade layout, dashboard, and profile pages — while Authgear handles authentication.

Install Breeze:

```sh
composer require laravel/breeze --dev
```

Run the installer to scaffold the resources:

```sh
php artisan breeze:install
```

During setup, select `blade` as the stack and leave the other options as default.

Next, add an `oauth_uid` column to the `users` table. This field stores a user's unique ID from Authgear after a successful login.

In Laravel 12, the users table is defined in `database/migrations/0001_01_01_000000_create_users_table.php` (this single file also creates the `sessions` and cache tables — there's no date-stamped users migration). Add the following line inside the `Schema::create('users', ...)` block:

```php
$table->string('oauth_uid')->nullable()->index();
```

Then add `oauth_uid` to the `$fillable` array in `app/Models/User.php` so the field can be mass-assigned:

```php
protected $fillable = [
    'name',
    'email',
    'oauth_uid',
    'password',
];
```

This example uses SQLite, so you don't need a database server. A fresh Laravel project already ships with `DB_CONNECTION=sqlite` in `.env`. Create the database file and run the migrations:

```bash
touch database/database.sqlite
php artisan migrate
```

#### Step 5: Add the Authgear Configuration

Keep your Authgear settings in a dedicated config file rather than calling `env()` from your controllers. This is what lets `php artisan config:cache` work in production — once the config is cached, `env()` returns `null` outside config files.

Create `config/authgear.php` with the following content:

```php
<?php

return [
    // Your Authgear project endpoint, e.g. https://my-project.authgear.cloud
    'project_url' => env('AUTHGEAR_PROJECT_URL', ''),

    'client_id' => env('AUTHGEAR_APP_CLIENT_ID', ''),
    'client_secret' => env('AUTHGEAR_APP_CLIENT_SECRET', ''),
    'redirect_uri' => env('AUTHGEAR_APP_REDIRECT_URI', ''),

    // OAuth 2.0 / OIDC scopes requested during authorization.
    'scopes' => env('AUTHGEAR_SCOPES', 'openid email profile'),

    // OIDC endpoints derived from the project URL.
    'authorize_endpoint' => env('AUTHGEAR_PROJECT_URL', '').'/oauth2/authorize',
    'token_endpoint' => env('AUTHGEAR_PROJECT_URL', '').'/oauth2/token',
    'userinfo_endpoint' => env('AUTHGEAR_PROJECT_URL', '').'/oauth2/userInfo',
    'end_session_endpoint' => env('AUTHGEAR_PROJECT_URL', '').'/oauth2/end_session',
];
```

Add your Authgear application's credentials to your project's `.env` file:

```
AUTHGEAR_PROJECT_URL=
AUTHGEAR_APP_CLIENT_ID=
AUTHGEAR_APP_CLIENT_SECRET=
AUTHGEAR_APP_REDIRECT_URI=http://localhost:8000/oauth/callback
```

{% hint style="info" %}
Your Authgear project URL is the hostname of any of your endpoint URLs. For a project with an authorization endpoint of `https://laravel-app.authgear.cloud/oauth2/authorize`, the project URL is `https://laravel-app.authgear.cloud`.
{% endhint %}

#### Step 6: Bind the OAuth Provider

We'll use the `league/oauth2-client` package to talk to Authgear's OAuth endpoints. Install it:

```sh
composer require league/oauth2-client
```

Bind a single configured `GenericProvider` in the service container so it can be injected into your controller. This keeps the OAuth setup in one place and makes the controller testable.

Open `app/Providers/AppServiceProvider.php` and add the binding to the `register()` method:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use League\OAuth2\Client\Provider\GenericProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(GenericProvider::class, function () {
            return new GenericProvider([
                'clientId' => config('authgear.client_id'),
                'clientSecret' => config('authgear.client_secret'),
                'redirectUri' => config('authgear.redirect_uri'),
                'urlAuthorize' => config('authgear.authorize_endpoint'),
                'urlAccessToken' => config('authgear.token_endpoint'),
                'urlResourceOwnerDetails' => config('authgear.userinfo_endpoint'),
            ]);
        });
    }

    public function boot(): void
    {
        //
    }
}
```

#### Step 7: Send the OAuth Authorization Request

In this step, you'll create the route that redirects users from your app to Authgear's authorization page, where they grant your app access to their account. If you've signed in to a site using Google before, you've seen an authorization page like this.

Create the controller that handles all OAuth operations:

```sh
php artisan make:controller OAuthController
```

Open `app/Http/Controllers/OAuthController.php`. Inject the provider through the constructor and add the `startAuthorization()` method:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;
use League\OAuth2\Client\Provider\Exception\IdentityProviderException;
use League\OAuth2\Client\Provider\GenericProvider;

class OAuthController extends Controller
{
    public function __construct(private GenericProvider $provider)
    {
    }

    public function startAuthorization(Request $request): RedirectResponse
    {
        $authorizationUrl = $this->provider->getAuthorizationUrl([
            'scope' => config('authgear.scopes'),
        ]);

        // Persist the state value to validate it on the callback (CSRF protection).
        $request->session()->put('oauth2state', $this->provider->getState());

        return redirect()->away($authorizationUrl);
    }
}
```

`startAuthorization()` stores the OAuth `state` value in the session before redirecting. You'll check it on the callback to protect against CSRF.

Now register the routes. Open `routes/web.php` and replace its contents with:

```php
<?php

use App\Http\Controllers\OAuthController;
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('index');
});

// Authgear OAuth 2.0 / OIDC flow.
Route::get('/login', [OAuthController::class, 'startAuthorization'])->name('login');
Route::get('/oauth/callback', [OAuthController::class, 'handleRedirect']);
Route::post('/logout', [OAuthController::class, 'logout'])->name('logout');

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');

Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});
```

The `/login` route is named `login` so Breeze's `auth` middleware redirects unauthenticated users to Authgear. You'll add the `handleRedirect()` and `logout()` methods in the next steps.

Because Authgear is now the only identity provider, you no longer need Breeze's local-auth routes. Delete `routes/auth.php` entirely — it defined the register, login, password-reset, and email-verification routes. The route file above already drops the `require __DIR__.'/auth.php';` line that Breeze added to `web.php`. You can also delete the matching controllers in `app/Http/Controllers/Auth/` and their Blade views if you want to keep the project tidy.

At this point, visiting `/login` should redirect to the Authgear authorization page.

<figure><img src="../../.gitbook/assets/authgear-authorization-page (1).png" alt=""><figcaption></figcaption></figure>

#### Step 8: Handle the Redirect

After a user authorizes your app, Authgear redirects back to `/oauth/callback` with an authorization code. The `handleRedirect()` method validates the request, exchanges the code for an access token, and fetches the user's info.

Add the `handleRedirect()` method to `OAuthController`:

```php
public function handleRedirect(Request $request): RedirectResponse
{
    $state = $request->query('state');
    $expectedState = $request->session()->pull('oauth2state');

    if (empty($state) || ! is_string($expectedState) || ! hash_equals($expectedState, $state)) {
        return redirect('/')->withErrors(['oauth' => 'Invalid OAuth state. Please try logging in again.']);
    }

    $code = $request->query('code');
    if (empty($code)) {
        return redirect('/')->withErrors(['oauth' => 'Authorization code missing.']);
    }

    try {
        $accessToken = $this->provider->getAccessToken('authorization_code', [
            'code' => $code,
        ]);

        $userInfo = $this->provider->getResourceOwner($accessToken)->toArray();
    } catch (IdentityProviderException $e) {
        report($e);

        return redirect('/')->withErrors(['oauth' => 'Failed to authenticate with Authgear.']);
    }

    if (empty($userInfo['sub'])) {
        return redirect('/')->withErrors(['oauth' => 'Authgear did not return a user identifier.']);
    }

    $user = $this->findOrCreateUser($userInfo);

    Auth::guard('web')->login($user);
    $request->session()->regenerate();

    return redirect()->intended('/dashboard');
}
```

A few things this method does that protect your app:

* **Validates `state`.** It compares the returned `state` against the value stored in Step 7 using `hash_equals`. A mismatch rejects the request, which blocks CSRF attacks against the callback.
* **Reads from the request, not `$_GET`.** Using `$request->query()` keeps the code consistent with the rest of Laravel and testable.
* **Fails gracefully.** Any error during the token exchange or userinfo call is reported and turned into a friendly redirect, instead of dumping an exception to the browser.

If you dump the `$userInfo` array (`dd($userInfo)`), you'll see the claims Authgear returns:

```json
[
  "custom_attributes" => []
  "email" => "users-email@gmail.com"
  "email_verified" => true
  "https://authgear.com/claims/user/can_reauthenticate" => true
  "https://authgear.com/claims/user/is_anonymous" => false
  "https://authgear.com/claims/user/is_verified" => true
  "sub" => "e1234323-f123-4b99-91d8-c2ca55a6a3dc"
  "updated_at" => 1683898685
]
```

The `sub` and `email_verified` claims drive the account-linking logic in the next step.

#### Step 9: Link the Authgear User to a Laravel Session

`handleRedirect()` calls `findOrCreateUser()` to map the Authgear user to a local Laravel user, so your app can start a normal authenticated session and guard protected routes.

Add the `findOrCreateUser()` method to `OAuthController`:

```php
private function findOrCreateUser(array $userInfo): User
{
    // Match on the stable Authgear subject identifier, never on email alone.
    $user = User::query()->where('oauth_uid', $userInfo['sub'])->first();

    if ($user) {
        return $user;
    }

    // Link to an existing local account by email ONLY when Authgear reports
    // the email as verified. Linking on an unverified email would allow
    // account takeover.
    if (! empty($userInfo['email']) && ($userInfo['email_verified'] ?? false) === true) {
        $existing = User::query()->where('email', $userInfo['email'])->first();

        if ($existing) {
            $existing->oauth_uid = $userInfo['sub'];
            $existing->save();

            return $existing;
        }
    }

    return User::create([
        'name' => $userInfo['email'] ?? $userInfo['sub'],
        'email' => $userInfo['email'] ?? null,
        'oauth_uid' => $userInfo['sub'],
        'password' => Hash::make(Str::random(40)),
    ]);
}
```

How this resolves a user:

1. **Match on `sub` first.** The Authgear subject identifier (`sub`) is stable and unique per user, so it's the reliable key. Email addresses can change or be reassigned.
2. **Link by email only when verified.** If no local user has this `sub` yet, link to an existing account by email — but only when `email_verified` is `true`. Linking on an unverified email would let an attacker claim someone else's account.
3. **Otherwise create a new user.** New users get a random local password they never use, since they always sign in through Authgear.

{% hint style="info" %}
This example doesn't store the access or refresh token in the session, and it doesn't request the `offline_access` scope. The scopes are `openid email profile`. The app reads the user's identity once at login and relies on the Laravel session from then on.
{% endhint %}

Find the complete `OAuthController` [here](https://github.com/authgear/authgear-example-laravel/blob/main/app/Http/Controllers/OAuthController.php).

Now run the app, open the landing page, and click the login link. You're redirected to the Authgear authorization page. After you authorize, Authgear sends you back to the callback route, and on success you land on the Breeze dashboard:

<figure><img src="../../.gitbook/assets/laravel-example-dashboad (1).png" alt=""><figcaption></figcaption></figure>

#### Step 10: Logout

To log a user out, clear the local Laravel session and then end the Authgear session so the user is fully signed out. Use the OIDC `end_session` endpoint rather than revoking a token — the app doesn't hold any tokens to revoke.

Add the `logout()` method to `OAuthController`:

```php
public function logout(Request $request): RedirectResponse
{
    Auth::guard('web')->logout();
    $request->session()->invalidate();
    $request->session()->regenerateToken();

    // If Authgear is configured, end its session too for a full sign-out.
    if (! empty(config('authgear.project_url'))) {
        return redirect()->away(config('authgear.end_session_endpoint'));
    }

    return redirect('/');
}
```

The `POST /logout` route is already wired up in `routes/web.php` from Step 7, so any Breeze logout button that posts to the `logout` route will trigger this method.

#### Verify It Works

The example repo ships a feature test in `tests/Feature/OAuthTest.php` that covers the state check, the code exchange, and account linking. Run the suite to confirm the flow behaves as expected:

```bash
php artisan test
```

### What's Next

Try enabling the different login methods on Authgear from the portal — 2FA, passwordless login, phone OTP, and more — without changing any code in your app.

The two hardening choices in this guide are worth keeping in any production integration: validating the OAuth `state` on the callback, and linking accounts by email only when Authgear reports the email as verified.

Find the complete code for the example app in the [Laravel Example GitHub repo](https://github.com/authgear/authgear-example-laravel).
