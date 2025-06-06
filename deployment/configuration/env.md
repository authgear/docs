---
description: Environment variables provides global configuration
---

# Environment Variables

## Main Server Environment Variables

Main server environment variable provides global configuration for main server.

### MAIN\_LISTEN\_ADDR

This sets the listen address of the main server. The default is `0.0.0.0:3000`.

### RESOLVER\_LISTEN\_ADDR

This sets the listen address of the resolver server. The default is `0.0.0.0:3001`.

### ADMIN\_LISTEN\_ADDR

This sets the listen address of the Admin API server. The default is `0.0.0.0:3002`.

### TLS\_CERT\_FILE\_PATH

This sets the file path of TLS certificate. It is only used when development mode is enabled. The default is `tls-cert.pem`.

### TLS\_KEY\_FILE\_PATH

This sets the file path of TLS private key. It is only used when development mode is enabled. The default is `tls-key.pem`.

### ADMIN\_API\_AUTH

This sets the authorization mode of the Admin API. Valid values are `jwt` and `none`. The default is `jwt`.

When the value is `jwt`, all requests to the Admin API must bear a valid JWT.

When the value is `none`, no authorization is needed. You must **NOT** use `none` in production unless you know the implied consequences.

### CONFIG\_SOURCE\_TYPE

This sets the type of the configuration. Valid values are `local_fs` and `kubernetes`. The default is `local_fs`.

### CONFIG\_SOURCE\_KUBECONFIG

This indicates the path to the `.kubeconfig` config file. It is only used when configuration type is `kubernetes`.

### CONFIG\_SOURCE\_KUBE\_NAMESPACE

This indicates the namespace where Kubernetes resources of all apps reside. It is only used when configuration type is `kubernetes`.

### CONFIG\_SOURCE\_WATCH

This indicates whether the configuration source would watch for changes and reload automatically. The default is `true`.

### CONFIG\_SOURCE\_DIRECTORY

This sets the path to app configuration directory file for local FS sources. The default is `.`.

### BUILTIN\_RESOURCE\_DIRECTORY

This sets the directory for built-in resource files. The default is `resources/authgear`.

### CUSTOM\_RESOURCE\_DIRECTORY

This sets the directory for customized resource files.

### STATIC\_ASSET\_SERVING\_ENABLED

This sets whether the bundled static asset should be served. Default is `true`. You should never modify it.

### STATIC\_ASSET\_DIR

This sets the filepath of the directory containing the bundled static asset. The default value of the provided Docker image does the right thing so you should never need to set it.

## Portal Environment Variable

Portal environment variable provides global configuration for Authegar portal.

### PORTAL\_LISTEN\_ADDR

This sets the listen address of the portal server. The default is `0.0.0.0:3003`.

### CONFIG\_SOURCE\_TYPE

This sets the type of the configuration. Valid values are `local_fs` and `kubernetes`. The default is `local_fs`.

### CONFIG\_SOURCE\_KUBECONFIG

This indicates the path to the `.kubeconfig` config file. It is only used when configuration type is `kubernetes`.

### CONFIG\_SOURCE\_KUBE\_NAMESPACE

This indicates the namespace where Kubernetes resources of all apps reside. It is only used when configuration type is `kubernetes`.

### CONFIG\_SOURCE\_WATCH

This indicates whether the configuration source would watch for changes and reload automatically. The default is `true`.

### CONFIG\_SOURCE\_DIRECTORY

This sets the path to app configuration directory file for local FS sources. The default is `.`.

### AUTHGEAR\_CLIENT\_ID

This sets the OAuth client ID for Authgear portal.

### AUTHGEAR\_ENDPOINT

This sets the OAuth endpoint for Authgear portal.

### AUTHGEAR\_APP\_ID

This sets the OAuth app ID for Authgear portal.

### ADMIN\_API\_TYPE

This sets the type of the admin API. The only supported value for now is `static`. The default is `static`, so you should never change it.

### ADMIN\_API\_ENDPOINT

This sets the endpoint of Admin API server. The default is `http://localhost:3002`.

### ADMIN\_API\_HOST\_TEMPLATE

This sets the host for tenant resolution. The default is `localhost:3002`.

### APP\_HOST\_SUFFIX

This sets the host suffix for Authgear portal.

### APP\_ID\_PATTERN

This sets the regular expression pattern for app ID validation. The defaults is `^[a-z0-9][a-z0-9-]{2,30}[a-z0-9]$`.

### APP\_KUBERNETES\_INGRESS\_TEMPLATE\_FILE

This sets the file of Kubernetes ingress template. It is only used when configuration type is `kubernetes`.

### APP\_KUBERNETES\_DEFAULT\_DOMAIN\_TLS\_CERT\_TYPE

This sets the TLS cert type for default domain. Valid values are `none`, `static`, and `cert-manager`. The default is `none`. It is only used when configuration type is `kubernetes`.

### APP\_KUBERNETES\_DEFAULT\_DOMAIN\_TLS\_CERT\_SECRET\_NAME

This sets the secret name for default domain. It is only used when configuration type is `kubernetes` and TLS cert type is `static`.

### APP\_KUBERNETES\_DEFAULT\_DOMAIN\_TLS\_CERT\_ISSUER\_KIND

This sets the issuer kind for default domain. It is only used when configuration type is `kubernetes` and TLS cert type is `cert-manager`.

### APP\_KUBERNETES\_DEFAULT\_DOMAIN\_TLS\_CERT\_ISSUER\_NAME

This sets the issuer name for default domain. It is only used when configuration type is `kubernetes` and TLS cert type is `cert-manager`.

### APP\_KUBERNETES\_CUSTOM\_DOMAIN\_TLS\_CERT\_TYPE

This sets the TLS cert type for custom domain. Valid values are `none`, `static`, and `cert-manager`. The default is `none`. It is only used when configuration type is `kubernetes`.

### APP\_KUBERNETES\_CUSTOM\_DOMAIN\_TLS\_CERT\_SECRET\_NAME

This sets the secret name for custom domain. It is only used when configuration type is `kubernetes` and TLS cert type is `static`.

### APP\_KUBERNETES\_CUSTOM\_DOMAIN\_TLS\_CERT\_ISSUER\_KIND

This sets the issuer kind for custom domain. It is only used when configuration type is `kubernetes` and TLS cert type is `cert-manager`.

### APP\_KUBERNETES\_CUSTOM\_DOMAIN\_TLS\_CERT\_ISSUER\_NAME

This sets the issuer name for custom domain. It is only used when configuration type is `kubernetes` and TLS cert type is `cert-manager`.

### APP\_BUILTIN\_RESOURCE\_DIRECTORY

This sets the directory for built-in resource files. The default is `resources/authgear`.

### APP\_CUSTOM\_RESOURCE\_DIRECTORY

This sets the directory for customized resource files.

### APP\_MAX\_OWNED\_APPS

This sets the maximum number of apps user owned. When the value is `-1`, owned apps limit is disabled. The default is `-1`.

### STATIC\_ASSET\_SERVING\_ENABLED

This sets whether the bundled static asset should be served. Default is `true`. You should never modify it.

### STATIC\_ASSET\_DIR

This sets the filepath of the directory containing the bundled static asset. The default value of the provided Docker image does the right thing so you should never need to set it.

### DATABASE\_URL

This sets the URL of backend database.

### DATABASE\_SCHEMA

This sets the database schema of backend database. The default is `public`.

### DATABASE\_MAX\_OPEN\_CONN

This sets the maximum open connections of backend database. The default is `2`.

### DATABASE\_MAX\_IDLE\_CONN

This sets the maximum idle connections of backend database. The default is `2`.

### DATABASE\_CONN\_MAX\_LIFETIME

This sets the maximum lifetime of backend database connection in seconds. The default is `1800`.

### DATABASE\_CONN\_MAX\_IDLE\_TIME

This sets the maximum idle time of backend database connection in seconds. The default is `300`.

### SMTP\_HOST

This sets the server host of SMTP server.

### SMTP\_PORT

This sets the server port of SMTP server.

### SMTP\_USERNAME

This sets the username of SMTP server.

### SMTP\_PASSWORD

This sets the password of SMTP server.

### SMTP\_MODE

This sets the SMTP mode. Valid values are `normal` and `ssl`. The default is `normal`.

### MAIL\_SENDER

This sets the sender field of admin invitation email.

### MAIL\_REPLY\_TO

This sets the reply to field of admin invitation email.

### PORTAL\_BUILTIN\_RESOURCE\_DIRECTORY

This sets the directory for built-in resource files. The default is `resources/portal`.

### PORTAL\_CUSTOM\_RESOURCE\_DIRECTORY

This sets the directory for customized resource files.

## Common Environment Variable

Common environment variable provides global configuration for both main server and Authgear portal.

### TRUST\_PROXY

This sets whether incoming HTTP headers such as `x-forwarded-host` can be trusted. If you deploy Authgear behind a reverse proxy capable of writing these headers, you should set the value to `true`. The default is `false`.

### DEV\_MODE

This sets whether Authgear should run in development mode. You should never need to set it. The default is `false`.

When development mode is enabled:

* TLS certificate is required, to enable secure cookies.
* All `Host` header values are allowed.
* External message sending (SMS/Email) is disabled; messages to send are logged instead.

### LOG\_LEVEL

This sets the global log level. Valid values are `debug`, `info`, `warn` and `error`. The default is `warn`.

### STATIC\_ASSET\_URL\_PREFIX

This sets the URL prefix of the bundled static asset. The default value includes commit hash so it is cache-friendly.

### SENTRY\_DSN

The sets the Sentry DSN, where errors/logs are reported to.

## TL;DR

The only environment variable you should be aware of is [TRUST\_PROXY](env.md#trust\_proxy).
