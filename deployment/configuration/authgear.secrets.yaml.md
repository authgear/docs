---
description: The secret configuration authgear.secrets.yaml
---

# authgear.secrets.yaml

This is the configuration file containing various secrets used in Authgear.

## JSON Schema

The configuration file is validated against the following JSON Schema:

```javascript
{
  "$defs": {
    "AdminAPIAuthKey": {
      "$ref": "#/$defs/JWS"
    },
    "AnalyticRedisCredentials": {
      "additionalProperties": false,
      "properties": {
        "redis_url": {
          "type": "string"
        }
      },
      "required": [
        "redis_url"
      ],
      "type": "object"
    },
    "AuditDatabaseCredentials": {
      "additionalProperties": false,
      "properties": {
        "database_schema": {
          "type": "string"
        },
        "database_url": {
          "type": "string"
        }
      },
      "required": [
        "database_url"
      ],
      "type": "object"
    },
    "CSRFKeyMaterials": {
      "$ref": "#/$defs/JWS"
    },
    "DatabaseCredentials": {
      "additionalProperties": false,
      "properties": {
        "database_schema": {
          "type": "string"
        },
        "database_url": {
          "type": "string"
        }
      },
      "required": [
        "database_url"
      ],
      "type": "object"
    },
    "ElasticsearchCredentials": {
      "additionalProperties": false,
      "properties": {
        "elasticsearch_url": {
          "type": "string"
        }
      },
      "required": [
        "elasticsearch_url"
      ],
      "type": "object"
    },
    "ImagesKeyMaterials": {
      "$ref": "#/$defs/JWS"
    },
    "JWK": {
      "properties": {
        "kid": {
          "type": "string"
        },
        "kty": {
          "type": "string"
        }
      },
      "required": [
        "kid",
        "kty"
      ],
      "type": "object"
    },
    "JWS": {
      "additionalProperties": false,
      "properties": {
        "keys": {
          "items": {
            "$ref": "#/$defs/JWK"
          },
          "minItems": 1,
          "type": "array"
        }
      },
      "required": [
        "keys"
      ],
      "type": "object"
    },
    "NexmoCredentials": {
      "additionalProperties": false,
      "properties": {
        "api_key": {
          "type": "string"
        },
        "api_secret": {
          "type": "string"
        }
      },
      "required": [
        "api_key",
        "api_secret"
      ],
      "type": "object"
    },
    "OAuthClientCredentials": {
      "additionalProperties": false,
      "properties": {
        "items": {
          "items": {
            "additionalProperties": false,
            "properties": {
              "alias": {
                "type": "string"
              },
              "client_secret": {
                "type": "string"
              }
            },
            "required": [
              "alias",
              "client_secret"
            ],
            "type": "object"
          },
          "type": "array"
        }
      },
      "required": [
        "items"
      ],
      "type": "object"
    },
    "OAuthKeyMaterials": {
      "$ref": "#/$defs/JWS"
    },
    "RedisCredentials": {
      "additionalProperties": false,
      "properties": {
        "redis_url": {
          "type": "string"
        }
      },
      "required": [
        "redis_url"
      ],
      "type": "object"
    },
    "SMTPMode": {
      "enum": [
        "normal",
        "ssl"
      ],
      "type": "string"
    },
    "SMTPServerCredentials": {
      "additionalProperties": false,
      "properties": {
        "host": {
          "type": "string"
        },
        "mode": {
          "$ref": "#/$defs/SMTPMode"
        },
        "password": {
          "type": "string"
        },
        "port": {
          "maximum": 65535,
          "minimum": 1,
          "type": "integer"
        },
        "username": {
          "type": "string"
        }
      },
      "required": [
        "host",
        "port",
        "username",
        "password"
      ],
      "type": "object"
    },
    "SecretConfig": {
      "additionalProperties": false,
      "properties": {
        "secrets": {
          "items": {
            "$ref": "#/$defs/SecretItem"
          },
          "type": "array"
        }
      },
      "required": [
        "secrets"
      ],
      "type": "object"
    },
    "SecretItem": {
      "additionalProperties": false,
      "allOf": [
        {
          "if": {
            "properties": {
              "key": {
                "const": "admin-api.auth"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/AdminAPIAuthKey"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "analytic.redis"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/AnalyticRedisCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "audit.db"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/AuditDatabaseCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "csrf"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/CSRFKeyMaterials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "db"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/DatabaseCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "elasticsearch"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/ElasticsearchCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "images"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/ImagesKeyMaterials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "mail.smtp"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/SMTPServerCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "oauth"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/OAuthKeyMaterials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "redis"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/RedisCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "sms.nexmo"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/NexmoCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "sms.twilio"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/TwilioCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "sso.oauth.client"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/OAuthClientCredentials"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "key": {
                "const": "webhook"
              }
            }
          },
          "then": {
            "properties": {
              "data": {
                "$ref": "#/$defs/WebhookKeyMaterials"
              }
            }
          }
        }
      ],
      "properties": {
        "data": {},
        "key": {
          "$ref": "#/$defs/SecretKey"
        }
      },
      "required": [
        "key",
        "data"
      ],
      "type": "object"
    },
    "SecretKey": {
      "enum": [
        "admin-api.auth",
        "analytic.redis",
        "audit.db",
        "csrf",
        "db",
        "elasticsearch",
        "images",
        "mail.smtp",
        "oauth",
        "redis",
        "sms.nexmo",
        "sms.twilio",
        "sso.oauth.client",
        "webhook"
      ],
      "type": "string"
    },
    "TwilioCredentials": {
      "additionalProperties": false,
      "properties": {
        "account_sid": {
          "type": "string"
        },
        "auth_token": {
          "type": "string"
        }
      },
      "required": [
        "account_sid",
        "auth_token"
      ],
      "type": "object"
    },
    "WebhookKeyMaterials": {
      "$ref": "#/$defs/JWS"
    }
  },
  "$ref": "#/$defs/SecretConfig"
}
```

## Structure

Secrets are placed under the key `secrets`. Each item has `key` and `data`. The valid values for `key` are listed below, where `data` is key-specific.

```yaml
secrets:
- key: well-known-key
  data: key-specific-data
```

Note that **ALL** secrets are required.

### admin-api.auth

`admin-api.auth` defines the JWK to verify Admin API token. It must be an RSA key.

```yaml
secrets:
- key: admin-api.auth
  data:
    keys:
    - kid: key_id
      kty: RSA
      # Other fields specific to RSA.
```

### db

`db` defines the database credentials. Only PostgreSQL database is supported.

```yaml
secrets:
- key: db
  data:
    database_url: postgresql://
    database_scheme: public
```

### audit.db

`audit.db` defines the database credentials of the instance for storing audit data. Only PostgreSQL database is supported.

```yaml
secrets:
- key: audit.db
  data:
    database_url: postgresql://
    database_scheme: public
```

### redis

`redis` defines the Redis credentials.

```yaml
secrets:
- key: redis
  data:
    redis_url: redis://username:password@localhost:6379/0
```

### analytic.redis

`analytic.redis` defines the Redis credentials of the Redis instance for storing analytics data.

```yaml
secrets:
- key: analytic.redis
  data:
    redis_url: redis://username:password@localhost:6379/0
```

### elasticsearch

`elasticsearch` defines the connection information of the Elasticsearch instance.

```yaml
secrets:
- key: elasticsearch
  data:
    elasticsearch_url: http://localhost:9200
```

### sso.oauth.client

`sso.oauth.client` defines the client secrets.

This is the place where you provide the client secrets of configured external OAuth providers.

```yaml
secrets:
- key: sso.oauth.client
  data:
    items:
    - alias: google
      client_secret: client_secret
    - alias: apple
      client_secret: private_key_in_pem_format
```

### mail.smtp

`mail.smtp` defines the SMTP credentials.

`mode` is either `ssl` or `normal`. Usually, you do not need to set it and the mode is inferred from the port.

```yaml
secrets:
- key: mail.smtp
  data:
    host: localhost
    port: 587
    mode: ssl
    username: username
    password: password
```

### sms.twilio

`sms.twilio` defines the Twilio credentials.

```yaml
secrets:
- key: sms.twilio
  data:
    account_sid: account_sid
    auth_token: auth_token
```

### sms.nexmo

`sms.nexmo` defines the Nexmo credentials.

```yaml
secrets:
- key: sms.nexmo
  data:
    api_key: api_key
    api_secret: api_secret
```

### jwt

`jwt` defines the JSON web key (JWK) to sign internal use, ephemeral JWT token. It must be an octet key.

```yaml
secrets:
- key: jwt
  data:
    keys:
    - kid: key_id
      kty: oct
      k: key
```

### oidc

`oidc` defines the JWK to sign ID tokens. It must be an RSA key.

```yaml
secrets:
- key: oidc
  data:
    keys:
    - kid: key_id
      kty: RSA
      # Other fields specific to RSA.
```

### csrf

`csrf` defines the symmetric key to generate a CSRF token. It must be an octet key.

The format shares with [jwt](authgear.secrets.yaml.md#jwt)

### webhook

`webhook` defines the symmetric key to sign webhook request body. It must be an octet key.

The format is shared with [jwt](authgear.secrets.yaml.md#jwt).
