---
description: Authentication for Spring Boot App with Authgear and OAuth2
---

# Java Spring Boot

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://raw.githubusercontent.com/authgear/docs/refs/heads/main/get-started/regular-web-app/java-spring-boot.md)

In this guide, you will learn how to add authentication to your Java Spring Boot application using [OAuth2](https://tools.ietf.org/html/rfc6749) with Authgear as the **Identity Provider (IdP)**.

### Learning objectives

You will learn the following:

* How to create an app on Authgear.
* How to enable Email based login.
* Add sign-up and login features to Spring Boot App.

### **Prerequisites**

Before you get started, you will need the following:

* Java 17 or higher.
* A **free Authgear account**. [Sign up](https://oursky.typeform.com/to/S5lvI8rN) if you don't have one already.

### Add login to your Spring Webapp

### Part 1: Configure Authgear

To use Authgear services, you’ll need to have an application set up in the Authgear [Dashboard](http://portal.authgear.com/). The Authgear application is where you will configure how you want authentication to work for the project you are developing.

#### Step 1: Configure an application

Use the interactive selector to create a new **Authgear OIDC Client application** or select an existing application that represents the project you want to integrate with.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae4853e73acf7a9284f8a8_Untitled%20(2)%20(1).png" alt=""><figcaption></figcaption></figure>

Every application in Authgear is assigned an alphanumeric, unique client ID that your application code will use to call Authgear APIs through the Spring Boot [OAuth 2 Client](https://docs.spring.io/spring-security/reference/reactive/oauth2/client/index.html). Note down the Authgear issuer (for example, https://example-auth.authgear.cloud/), CLIENT ID, CLIENT SECRET, and OpenID endpoints from the output. You will use these values in the next step for the client app config.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae485914ea3016deaaad18_Untitled%20(3)%20(1).png" alt=""><figcaption></figcaption></figure>

#### Step 2: Configure **Redirect URI**

A **Redirect URI** is a URL in your application that you would like Authgear to redirect users to after they have authenticated. In our case, it will be a home page for our Spring Boot App. If not set, users will not be returned to your application after they log in.

To follow the example in this post, add the following URL as a redirect URI:

```
http://localhost:8080/login/oauth2/code/authgear
```

#### Step 3: Choose a Login method

After you create the Authgear app, you choose how users need to **authenticate on the login page**. From the “Authentication” tab, navigate to “Login Methods”, you can choose a **login method** from various options including by email, mobile, or social, just using a username or the custom method you specify. For this demo, we choose the **Email+Passwordless** approach where our users are asked to register an account and log in by using their emails. They will receive a One-time password (OTP) to their emails and verify the code to use the app.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae48ba190172f7f9f9cc0e_Untitled%20(4)%20(1).png" alt=""><figcaption></figcaption></figure>

### Part 2: Configure Spring Boot application

#### Step 1: Add Spring dependencies

To create a new Spring Boot application you use the [Spring Initializr](https://start.spring.io/). Then you add dependencies to pom.xml file such as [spring-boot-starter-oauth2-client](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-oauth2-client) starter provides all the Spring Security dependencies needed to add authentication to your web application and Thymeleaf is used just to build a single page UI.

```

  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-oauth2-client</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-thymeleaf</artifactId>
      </dependency>
      <dependency>
          <groupId>org.thymeleaf.extras</groupId>
          <artifactId>thymeleaf-extras-springsecurity6</artifactId>
          <version>3.1.1.RELEASE</version>
      </dependency>
  </dependencies>
  
```

#### Step 2: Configure OIDC authentication with Authgear

Spring Security makes it easy to configure your application for authentication with OIDC providers such as Authgear. We need to add the client credentials to the **application.properties** file with your Auhgear provider configuration. You can use the sample below and replace properties with the values from your Authgear app:

```properties

spring.security.oauth2.client.registration.authgear.client-id={your-client-id}
spring.security.oauth2.client.registration.authgear.client-secret={your-client-secret}
spring.security.oauth2.client.registration.authgear.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.authgear.scope=openid,offline_access
spring.security.oauth2.client.registration.authgear.redirect-uri=http://localhost:8080/login/oauth2/code/authgear/
spring.security.oauth2.client.provider.authgear.token-uri=https://{DOMAIN}/oauth2/token
spring.security.oauth2.client.provider.authgear.authorization-uri=https://{DOMAIN}/oauth2/authorize

# To logout from the app
authgear.oauth2.end-session-endpoint=https://{DOMAIN}/oauth2/end_session
  
```

#### Step 3: Add login to your application

To enable user login with Authgear, create a class that will provide an instance of **SecurityFilterChain**, add the `@EnableMethodSecurity` annotation, and override the necessary method:

```java

@Configuration
@EnableMethodSecurity(securedEnabled = true)
public class SecurityConfig {

    @Value("${authgear.oauth2.end-session-endpoint}")
    private String endSessionEndpoint;

    @Bean
    public SecurityFilterChain configure(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests((requests) -> requests
                // allow anonymous access to the root page
                .requestMatchers("/").permitAll()
                // authenticate all other requests
                .anyRequest().authenticated())
            // enable OAuth2/OIDC
            .oauth2Login(withDefaults())
            // configure logout handler
            .logout(logout -> logout.logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/")
                .addLogoutHandler(oidcLogoutHandler()));
        return http.build();
    }

    LogoutHandler oidcLogoutHandler() {
        return (request, response, authentication) -> {
            try {
                response.sendRedirect(endSessionEndpoint);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        };
    }
}
  
```

#### Step 4: Add the front page

We create a simple home.html page using Thymeleaf templates. When a user opens the page running on http://localhost:8080/, we show the page with buttons for login or logout:

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae4b0a99ffe304fc9e50f4_Untitled%20(11).png" alt=""><figcaption></figcaption></figure>

#### Step 5: Add controller

Next, we create a controller class to handle the incoming request. This controller renders the home.html page. When the user authenticates, the application retrieves the user's profile information attributes to render the page.

```

@Controller
public class HomeController {
    @GetMapping("/")
    String home() {
        return "home";
    }
}
  
```

#### Step 6: Run the Application

To run the application, you can execute the `mvn spring-boot:run` goal. Or run from your editor the main ExampleApplication.java file. The sample application will be available at http://localhost:8080/.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae61c4c93bc48ef731785f_Untitled%20(13).png" alt=""><figcaption></figcaption></figure>

Click on the **Login** button to be redirected to the Authgear login page.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae61ca49cc298d411f03d6_Untitled%20(14).png" alt=""><figcaption></figcaption></figure>

You can also customize the login page UI view from the Authgear Portal. After you sign up, you will receive an OTP code in your email to verify your identity.

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae61e40046a0f0dbeff9d0_Untitled%20(15).png" alt=""><figcaption></figcaption></figure>

And log into your new account, you will be redirected back to the home page:

<figure><img src="https://uploads-ssl.webflow.com/60658b47b03f0c77e8c14884/64ae61eb29d7e0cb537f5f01_Untitled%20(16).png" alt=""><figcaption></figcaption></figure>

You have successfully configured a Spring Boot application to use Authgear for authentication. Now users can sign up for a new account, log in, and log out. The full source code of the examples can be found [on GitHub](https://github.com/Boburmirzo/authgear-spring-oauth2-example).

### Next steps

There is so much more you can do with Authgear. Explore other means of login methods such as using [Magic links](https://docs.authgear.com/strategies/email-login-link) in an email, [social logins](https://docs.authgear.com/strategies/how-to-setup-sso-integrations), or [WhatsApp OTP](https://docs.authgear.com/strategies/whatsapp-otp-login). For the current application, you can also [add more users](https://docs.authgear.com/strategies/user-identity-and-authenticator) from the Authgear portal.
