---
description: The app configuration authgear.yaml
---

# authgear.yaml

This is the main configuration file affecting every aspect of Authgear.

## JSON Schema

The configuration file is validated against the following JSON Schema:

```javascript
{
  "$defs": {
    "AccessControlLevelString": {
      "enum": [
        "hidden",
        "readonly",
        "readwrite"
      ],
      "type": "string"
    },
    "AccountDeletionConfig": {
      "additionalProperties": false,
      "properties": {
        "grace_period_days": {
          "$ref": "#/$defs/DurationDays",
          "maximum": 180,
          "minimum": 1
        },
        "scheduled_by_end_user_enabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "AppConfig": {
      "additionalProperties": false,
      "properties": {
        "account_deletion": {
          "$ref": "#/$defs/AccountDeletionConfig"
        },
        "authentication": {
          "$ref": "#/$defs/AuthenticationConfig"
        },
        "authenticator": {
          "$ref": "#/$defs/AuthenticatorConfig"
        },
        "forgot_password": {
          "$ref": "#/$defs/ForgotPasswordConfig"
        },
        "google_tag_manager": {
          "$ref": "#/$defs/GoogleTagManagerConfig"
        },
        "hook": {
          "$ref": "#/$defs/HookConfig"
        },
        "http": {
          "$ref": "#/$defs/HTTPConfig"
        },
        "id": {
          "type": "string"
        },
        "identity": {
          "$ref": "#/$defs/IdentityConfig"
        },
        "localization": {
          "$ref": "#/$defs/LocalizationConfig"
        },
        "messaging": {
          "$ref": "#/$defs/MessagingConfig"
        },
        "oauth": {
          "$ref": "#/$defs/OAuthConfig"
        },
        "session": {
          "$ref": "#/$defs/SessionConfig"
        },
        "ui": {
          "$ref": "#/$defs/UIConfig"
        },
        "user_profile": {
          "$ref": "#/$defs/UserProfileConfig"
        },
        "verification": {
          "$ref": "#/$defs/VerificationConfig"
        },
        "welcome_message": {
          "$ref": "#/$defs/WelcomeMessageConfig"
        }
      },
      "required": [
        "id",
        "http"
      ],
      "type": "object"
    },
    "AuthenticationConfig": {
      "additionalProperties": false,
      "properties": {
        "device_token": {
          "$ref": "#/$defs/DeviceTokenConfig"
        },
        "identities": {
          "items": {
            "$ref": "#/$defs/IdentityType"
          },
          "type": "array",
          "uniqueItems": true
        },
        "primary_authenticators": {
          "items": {
            "$ref": "#/$defs/PrimaryAuthenticatorType"
          },
          "type": "array",
          "uniqueItems": true
        },
        "public_signup_disabled": {
          "type": "boolean"
        },
        "recovery_code": {
          "$ref": "#/$defs/RecoveryCodeConfig"
        },
        "secondary_authentication_mode": {
          "$ref": "#/$defs/SecondaryAuthenticationMode"
        },
        "secondary_authenticators": {
          "items": {
            "$ref": "#/$defs/SecondaryAuthenticatorType"
          },
          "type": "array",
          "uniqueItems": true
        }
      },
      "type": "object"
    },
    "AuthenticatorConfig": {
      "additionalProperties": false,
      "properties": {
        "oob_otp": {
          "$ref": "#/$defs/AuthenticatorOOBConfig"
        },
        "password": {
          "$ref": "#/$defs/AuthenticatorPasswordConfig"
        },
        "totp": {
          "$ref": "#/$defs/AuthenticatorTOTPConfig"
        }
      },
      "type": "object"
    },
    "AuthenticatorOOBConfig": {
      "additionalProperties": false,
      "properties": {
        "email": {
          "$ref": "#/$defs/AuthenticatorOOBEmailConfig"
        },
        "sms": {
          "$ref": "#/$defs/AuthenticatorOOBSMSConfig"
        }
      },
      "type": "object"
    },
    "AuthenticatorOOBEmailConfig": {
      "additionalProperties": false,
      "properties": {
        "maximum": {
          "type": "integer"
        }
      },
      "type": "object"
    },
    "AuthenticatorOOBSMSConfig": {
      "additionalProperties": false,
      "properties": {
        "maximum": {
          "type": "integer"
        }
      },
      "type": "object"
    },
    "AuthenticatorPasswordConfig": {
      "additionalProperties": false,
      "properties": {
        "force_change": {
          "type": "boolean"
        },
        "policy": {
          "$ref": "#/$defs/PasswordPolicyConfig"
        }
      },
      "type": "object"
    },
    "AuthenticatorTOTPConfig": {
      "additionalProperties": false,
      "properties": {
        "maximum": {
          "type": "integer"
        }
      },
      "type": "object"
    },
    "BiometricConfig": {
      "additionalProperties": false,
      "properties": {
        "list_enabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "BlockingHookHandlersConfig": {
      "additionalProperties": false,
      "properties": {
        "event": {
          "enum": [
            "user.pre_create",
            "user.profile.pre_update",
            "user.pre_schedule_deletion"
          ],
          "type": "string"
        },
        "url": {
          "format": "uri",
          "type": "string"
        }
      },
      "required": [
        "event",
        "url"
      ],
      "type": "object"
    },
    "CustomAttributesAttributeConfig": {
      "allOf": [
        {
          "if": {
            "properties": {
              "type": {
                "const": "number"
              }
            }
          },
          "then": {
            "properties": {
              "maximum": {
                "type": "number"
              },
              "minimum": {
                "type": "number"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "const": "integer"
              }
            }
          },
          "then": {
            "properties": {
              "maximum": {
                "type": "integer"
              },
              "minimum": {
                "type": "integer"
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "const": "enum"
              }
            }
          },
          "then": {
            "properties": {
              "enum": {
                "items": {
                  "minLength": 1,
                  "pattern": "^[a-zA-Z0-9_]*$",
                  "type": "string"
                },
                "minItems": 1,
                "type": "array",
                "uniqueItems": true
              }
            },
            "required": [
              "enum"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "not": {
                  "enum": [
                    "number",
                    "integer",
                    "enum"
                  ]
                }
              }
            }
          },
          "then": true
        }
      ],
      "properties": {
        "access_control": {
          "$ref": "#/$defs/UserProfileAttributesAccessControl"
        },
        "id": {
          "minLength": 1,
          "type": "string"
        },
        "pointer": {
          "format": "x_custom_attribute_pointer",
          "not": {
            "enum": [
              "/iss",
              "/sub",
              "/aud",
              "/exp",
              "/nbf",
              "/iat",
              "/jti",
              "/sub",
              "/email",
              "/email_verified",
              "/phone_number",
              "/phone_number_verified",
              "/preferred_username",
              "/family_name",
              "/given_name",
              "/picture",
              "/gender",
              "/birthdate",
              "/zoneinfo",
              "/locale",
              "/name",
              "/nickname",
              "/middle_name",
              "/profile",
              "/website",
              "/address",
              "/updated_at"
            ]
          },
          "type": "string"
        },
        "type": {
          "enum": [
            "string",
            "number",
            "integer",
            "enum",
            "phone_number",
            "email",
            "url",
            "country_code"
          ],
          "type": "string"
        }
      },
      "required": [
        "id",
        "pointer",
        "type"
      ],
      "type": "object"
    },
    "CustomAttributesConfig": {
      "additionalProperties": false,
      "properties": {
        "attributes": {
          "items": {
            "$ref": "#/$defs/CustomAttributesAttributeConfig"
          },
          "type": "array"
        }
      },
      "type": "object"
    },
    "DeviceTokenConfig": {
      "additionalProperties": false,
      "properties": {
        "disabled": {
          "type": "boolean"
        },
        "expire_in_days": {
          "$ref": "#/$defs/DurationDays"
        }
      },
      "type": "object"
    },
    "DurationDays": {
      "type": "integer"
    },
    "DurationSeconds": {
      "type": "integer"
    },
    "ForgotPasswordConfig": {
      "additionalProperties": false,
      "properties": {
        "enabled": {
          "type": "boolean"
        },
        "reset_code_expiry_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        }
      },
      "type": "object"
    },
    "GoogleTagManagerConfig": {
      "additionalProperties": false,
      "properties": {
        "container_id": {
          "format": "google_tag_manager_container_id",
          "type": "string"
        }
      },
      "type": "object"
    },
    "HTTPConfig": {
      "additionalProperties": false,
      "properties": {
        "allowed_origins": {
          "items": {
            "format": "http_origin_spec",
            "minLength": 1,
            "type": "string"
          },
          "type": "array"
        },
        "cookie_domain": {
          "type": "string"
        },
        "cookie_prefix": {
          "type": "string"
        },
        "public_origin": {
          "format": "http_origin",
          "type": "string"
        }
      },
      "required": [
        "public_origin"
      ],
      "type": "object"
    },
    "HookConfig": {
      "additionalProperties": false,
      "properties": {
        "blocking_handlers": {
          "items": {
            "$ref": "#/$defs/BlockingHookHandlersConfig"
          },
          "type": "array"
        },
        "non_blocking_handlers": {
          "items": {
            "$ref": "#/$defs/NonBlockingHookHandlersConfig"
          },
          "type": "array"
        },
        "sync_hook_timeout_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        },
        "sync_hook_total_timeout_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        }
      },
      "type": "object"
    },
    "ISO31661Alpha2": {
      "enum": [
        "AD",
        "AE",
        "AF",
        "AG",
        "AI",
        "AL",
        "AM",
        "AO",
        "AR",
        "AS",
        "AT",
        "AU",
        "AW",
        "AX",
        "AZ",
        "BA",
        "BB",
        "BD",
        "BE",
        "BF",
        "BG",
        "BH",
        "BI",
        "BJ",
        "BL",
        "BM",
        "BN",
        "BO",
        "BQ",
        "BR",
        "BS",
        "BT",
        "BW",
        "BY",
        "BZ",
        "CA",
        "CC",
        "CD",
        "CF",
        "CG",
        "CH",
        "CI",
        "CK",
        "CL",
        "CM",
        "CN",
        "CO",
        "CR",
        "CU",
        "CV",
        "CW",
        "CX",
        "CY",
        "CZ",
        "DE",
        "DJ",
        "DK",
        "DM",
        "DO",
        "DZ",
        "EC",
        "EE",
        "EG",
        "EH",
        "ER",
        "ES",
        "ET",
        "FI",
        "FJ",
        "FK",
        "FM",
        "FO",
        "FR",
        "GA",
        "GB",
        "GD",
        "GE",
        "GF",
        "GG",
        "GH",
        "GI",
        "GL",
        "GM",
        "GN",
        "GP",
        "GQ",
        "GR",
        "GT",
        "GU",
        "GW",
        "GY",
        "HK",
        "HN",
        "HR",
        "HT",
        "HU",
        "ID",
        "IE",
        "IL",
        "IM",
        "IN",
        "IO",
        "IQ",
        "IR",
        "IS",
        "IT",
        "JE",
        "JM",
        "JO",
        "JP",
        "KE",
        "KG",
        "KH",
        "KI",
        "KM",
        "KN",
        "KP",
        "KR",
        "KW",
        "KY",
        "KZ",
        "LA",
        "LB",
        "LC",
        "LI",
        "LK",
        "LR",
        "LS",
        "LT",
        "LU",
        "LV",
        "LY",
        "MA",
        "MC",
        "MD",
        "ME",
        "MF",
        "MG",
        "MH",
        "MK",
        "ML",
        "MM",
        "MN",
        "MO",
        "MP",
        "MQ",
        "MR",
        "MS",
        "MT",
        "MU",
        "MV",
        "MW",
        "MX",
        "MY",
        "MZ",
        "NA",
        "NC",
        "NE",
        "NF",
        "NG",
        "NI",
        "NL",
        "NO",
        "NP",
        "NR",
        "NU",
        "NZ",
        "OM",
        "PA",
        "PE",
        "PF",
        "PG",
        "PH",
        "PK",
        "PL",
        "PM",
        "PR",
        "PS",
        "PT",
        "PW",
        "PY",
        "QA",
        "RE",
        "RO",
        "RS",
        "RU",
        "RW",
        "SA",
        "SB",
        "SC",
        "SD",
        "SE",
        "SG",
        "SH",
        "SI",
        "SJ",
        "SK",
        "SL",
        "SM",
        "SN",
        "SO",
        "SR",
        "SS",
        "ST",
        "SV",
        "SX",
        "SY",
        "SZ",
        "TC",
        "TD",
        "TG",
        "TH",
        "TJ",
        "TK",
        "TL",
        "TM",
        "TN",
        "TO",
        "TR",
        "TT",
        "TV",
        "TW",
        "TZ",
        "UA",
        "UG",
        "US",
        "UY",
        "UZ",
        "VA",
        "VC",
        "VE",
        "VG",
        "VI",
        "VN",
        "VU",
        "WF",
        "WS",
        "XK",
        "YE",
        "YT",
        "ZA",
        "ZM",
        "ZW"
      ],
      "type": "string"
    },
    "IdentityConfig": {
      "additionalProperties": false,
      "properties": {
        "biometric": {
          "$ref": "#/$defs/BiometricConfig"
        },
        "login_id": {
          "$ref": "#/$defs/LoginIDConfig"
        },
        "oauth": {
          "$ref": "#/$defs/OAuthSSOConfig"
        },
        "on_conflict": {
          "$ref": "#/$defs/IdentityConflictConfig"
        }
      },
      "type": "object"
    },
    "IdentityConflictConfig": {
      "additionalProperties": false,
      "properties": {
        "promotion": {
          "$ref": "#/$defs/PromotionConflictBehavior"
        }
      },
      "type": "object"
    },
    "IdentityType": {
      "enum": [
        "login_id",
        "oauth",
        "anonymous",
        "biometric"
      ],
      "type": "string"
    },
    "LocalizationConfig": {
      "additionalProperties": false,
      "properties": {
        "fallback_language": {
          "format": "bcp47",
          "type": "string"
        },
        "supported_languages": {
          "items": {
            "format": "bcp47",
            "type": "string"
          },
          "minItems": 1,
          "type": "array",
          "uniqueItems": true
        }
      },
      "type": "object"
    },
    "LoginIDConfig": {
      "additionalProperties": false,
      "properties": {
        "keys": {
          "items": {
            "$ref": "#/$defs/LoginIDKeyConfig"
          },
          "type": "array"
        },
        "types": {
          "$ref": "#/$defs/LoginIDTypesConfig"
        }
      },
      "type": "object"
    },
    "LoginIDEmailConfig": {
      "additionalProperties": false,
      "allOf": [
        {
          "if": {
            "properties": {
              "domain_blocklist_enabled": {
                "enum": [
                  true
                ]
              }
            },
            "required": [
              "domain_blocklist_enabled"
            ]
          },
          "then": {
            "properties": {
              "domain_allowlist_enabled": {
                "enum": [
                  false
                ]
              }
            }
          }
        },
        {
          "if": {
            "properties": {
              "block_free_email_provider_domains": {
                "enum": [
                  true
                ]
              }
            },
            "required": [
              "block_free_email_provider_domains"
            ]
          },
          "then": {
            "properties": {
              "domain_blocklist_enabled": {
                "enum": [
                  true
                ]
              }
            },
            "required": [
              "domain_blocklist_enabled"
            ]
          }
        }
      ],
      "properties": {
        "block_free_email_provider_domains": {
          "type": "boolean"
        },
        "block_plus_sign": {
          "type": "boolean"
        },
        "case_sensitive": {
          "type": "boolean"
        },
        "domain_allowlist_enabled": {
          "type": "boolean"
        },
        "domain_blocklist_enabled": {
          "type": "boolean"
        },
        "ignore_dot_sign": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "LoginIDKeyConfig": {
      "additionalProperties": false,
      "properties": {
        "key": {
          "type": "string"
        },
        "max_length": {
          "type": "integer"
        },
        "modify_disabled": {
          "type": "boolean"
        },
        "type": {
          "$ref": "#/$defs/LoginIDKeyType"
        }
      },
      "required": [
        "type"
      ],
      "type": "object"
    },
    "LoginIDKeyType": {
      "enum": [
        "email",
        "phone",
        "username"
      ],
      "type": "string"
    },
    "LoginIDTypesConfig": {
      "additionalProperties": false,
      "properties": {
        "email": {
          "$ref": "#/$defs/LoginIDEmailConfig"
        },
        "username": {
          "$ref": "#/$defs/LoginIDUsernameConfig"
        }
      },
      "type": "object"
    },
    "LoginIDUsernameConfig": {
      "additionalProperties": false,
      "properties": {
        "ascii_only": {
          "type": "boolean"
        },
        "block_reserved_usernames": {
          "type": "boolean"
        },
        "case_sensitive": {
          "type": "boolean"
        },
        "exclude_keywords_enabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "MessagingConfig": {
      "additionalProperties": false,
      "properties": {
        "sms_provider": {
          "$ref": "#/$defs/SMSProvider"
        }
      },
      "type": "object"
    },
    "NonBlockingHookHandlersConfig": {
      "additionalProperties": false,
      "properties": {
        "events": {
          "items": {
            "enum": [
              "*",
              "user.created",
              "user.authenticated",
              "user.profile.updated",
              "user.disabled",
              "user.reenabled",
              "user.anonymous.promoted",
              "user.deletion_scheduled",
              "user.deletion_unscheduled",
              "user.deleted",
              "identity.email.added",
              "identity.email.removed",
              "identity.email.updated",
              "identity.phone.added",
              "identity.phone.removed",
              "identity.phone.updated",
              "identity.username.added",
              "identity.username.removed",
              "identity.username.updated",
              "identity.oauth.connected",
              "identity.oauth.disconnected"
            ],
            "type": "string"
          },
          "type": "array"
        },
        "url": {
          "format": "uri",
          "type": "string"
        }
      },
      "required": [
        "events",
        "url"
      ],
      "type": "object"
    },
    "OAuthClaimConfig": {
      "additionalProperties": false,
      "properties": {
        "assume_verified": {
          "type": "boolean"
        },
        "required": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "OAuthClaimsConfig": {
      "additionalProperties": false,
      "properties": {
        "email": {
          "$ref": "#/$defs/OAuthClaimConfig"
        }
      },
      "type": "object"
    },
    "OAuthClientConfig": {
      "additionalProperties": false,
      "properties": {
        "access_token_lifetime_seconds": {
          "$ref": "#/$defs/DurationSeconds",
          "minimum": 300
        },
        "client_id": {
          "type": "string"
        },
        "client_uri": {
          "format": "uri",
          "type": "string"
        },
        "grant_types": {
          "items": {
            "type": "string"
          },
          "type": "array"
        },
        "is_first_party": {
          "type": "boolean"
        },
        "issue_jwt_access_token": {
          "type": "boolean"
        },
        "name": {
          "type": "string"
        },
        "post_logout_redirect_uris": {
          "items": {
            "format": "uri",
            "type": "string"
          },
          "type": "array"
        },
        "redirect_uris": {
          "items": {
            "format": "uri",
            "type": "string"
          },
          "minItems": 1,
          "type": "array"
        },
        "refresh_token_idle_timeout_enabled": {
          "type": "boolean"
        },
        "refresh_token_idle_timeout_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        },
        "refresh_token_lifetime_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        },
        "response_types": {
          "items": {
            "type": "string"
          },
          "type": "array"
        }
      },
      "required": [
        "name",
        "client_id",
        "redirect_uris"
      ],
      "type": "object"
    },
    "OAuthConfig": {
      "additionalProperties": false,
      "properties": {
        "clients": {
          "items": {
            "$ref": "#/$defs/OAuthClientConfig"
          },
          "type": "array"
        }
      },
      "type": "object"
    },
    "OAuthSSOConfig": {
      "additionalProperties": false,
      "properties": {
        "providers": {
          "items": {
            "$ref": "#/$defs/OAuthSSOProviderConfig"
          },
          "type": "array"
        }
      },
      "type": "object"
    },
    "OAuthSSOProviderConfig": {
      "additionalProperties": false,
      "allOf": [
        {
          "if": {
            "properties": {
              "type": {
                "const": "apple"
              }
            }
          },
          "then": {
            "required": [
              "key_id",
              "team_id"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "const": "azureadv2"
              }
            }
          },
          "then": {
            "required": [
              "tenant"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "const": "wechat"
              }
            }
          },
          "then": {
            "required": [
              "app_type",
              "account_id"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "const": "adfs"
              }
            }
          },
          "then": {
            "required": [
              "discovery_document_endpoint"
            ]
          }
        },
        {
          "if": {
            "properties": {
              "type": {
                "const": "azureadb2c"
              }
            }
          },
          "then": {
            "required": [
              "tenant",
              "policy"
            ]
          }
        }
      ],
      "properties": {
        "account_id": {
          "format": "wechat_account_id",
          "type": "string"
        },
        "alias": {
          "type": "string"
        },
        "app_type": {
          "$ref": "#/$defs/OAuthSSOWeChatAppType"
        },
        "claims": {
          "$ref": "#/$defs/OAuthClaimsConfig"
        },
        "client_id": {
          "type": "string"
        },
        "discovery_document_endpoint": {
          "format": "uri",
          "type": "string"
        },
        "is_sandbox_account": {
          "type": "boolean"
        },
        "key_id": {
          "type": "string"
        },
        "modify_disabled": {
          "type": "boolean"
        },
        "policy": {
          "type": "string"
        },
        "team_id": {
          "type": "string"
        },
        "tenant": {
          "type": "string"
        },
        "type": {
          "$ref": "#/$defs/OAuthSSOProviderType"
        },
        "wechat_redirect_uris": {
          "items": {
            "format": "uri",
            "type": "string"
          },
          "type": "array"
        }
      },
      "required": [
        "alias",
        "type",
        "client_id"
      ],
      "type": "object"
    },
    "OAuthSSOProviderType": {
      "enum": [
        "google",
        "facebook",
        "github",
        "linkedin",
        "azureadv2",
        "azureadb2c",
        "adfs",
        "apple",
        "wechat"
      ],
      "type": "string"
    },
    "OAuthSSOWeChatAppType": {
      "enum": [
        "mobile",
        "web"
      ],
      "type": "string"
    },
    "PasswordPolicyConfig": {
      "additionalProperties": false,
      "properties": {
        "digit_required": {
          "type": "boolean"
        },
        "excluded_keywords": {
          "items": {
            "type": "string"
          },
          "type": "array"
        },
        "history_days": {
          "$ref": "#/$defs/DurationDays"
        },
        "history_size": {
          "type": "integer"
        },
        "lowercase_required": {
          "type": "boolean"
        },
        "min_length": {
          "minimum": 1,
          "type": "integer"
        },
        "minimum_guessable_level": {
          "type": "integer"
        },
        "symbol_required": {
          "type": "boolean"
        },
        "uppercase_required": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "PhoneInputConfig": {
      "additionalProperties": false,
      "properties": {
        "allowlist": {
          "items": {
            "$ref": "#/$defs/ISO31661Alpha2"
          },
          "minItems": 1,
          "type": "array"
        },
        "pinned_list": {
          "items": {
            "$ref": "#/$defs/ISO31661Alpha2"
          },
          "type": "array"
        },
        "preselect_by_ip_disabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "PrimaryAuthenticatorType": {
      "enum": [
        "password",
        "oob_otp_email",
        "oob_otp_sms"
      ],
      "type": "string"
    },
    "PromotionConflictBehavior": {
      "enum": [
        "error",
        "login"
      ],
      "type": "string"
    },
    "RecoveryCodeConfig": {
      "additionalProperties": false,
      "properties": {
        "count": {
          "maximum": 50,
          "minimum": 10,
          "type": "integer"
        },
        "list_enabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "SMSProvider": {
      "enum": [
        "nexmo",
        "twilio"
      ],
      "type": "string"
    },
    "SecondaryAuthenticationMode": {
      "enum": [
        "disabled",
        "if_exists",
        "required"
      ],
      "type": "string"
    },
    "SecondaryAuthenticatorType": {
      "enum": [
        "password",
        "oob_otp_email",
        "oob_otp_sms",
        "totp"
      ],
      "type": "string"
    },
    "SessionConfig": {
      "additionalProperties": false,
      "properties": {
        "cookie_non_persistent": {
          "type": "boolean"
        },
        "idle_timeout_enabled": {
          "type": "boolean"
        },
        "idle_timeout_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        },
        "lifetime_seconds": {
          "$ref": "#/$defs/DurationSeconds"
        }
      },
      "type": "object"
    },
    "StandardAttributesAccessControlConfig": {
      "additionalProperties": false,
      "properties": {
        "access_control": {
          "$ref": "#/$defs/UserProfileAttributesAccessControl"
        },
        "pointer": {
          "enum": [
            "/email",
            "/phone_number",
            "/preferred_username",
            "/family_name",
            "/given_name",
            "/picture",
            "/gender",
            "/birthdate",
            "/zoneinfo",
            "/locale",
            "/name",
            "/nickname",
            "/middle_name",
            "/profile",
            "/website",
            "/address"
          ],
          "format": "json-pointer",
          "type": "string"
        }
      },
      "type": "object"
    },
    "StandardAttributesConfig": {
      "additionalProperties": false,
      "properties": {
        "access_control": {
          "items": {
            "$ref": "#/$defs/StandardAttributesAccessControlConfig"
          },
          "type": "array"
        },
        "population": {
          "$ref": "#/$defs/StandardAttributesPopulationConfig"
        }
      },
      "type": "object"
    },
    "StandardAttributesPopulationConfig": {
      "additionalProperties": false,
      "properties": {
        "strategy": {
          "enum": [
            "none",
            "on_signup"
          ],
          "type": "string"
        }
      },
      "type": "object"
    },
    "UIConfig": {
      "additionalProperties": false,
      "properties": {
        "dark_theme_disabled": {
          "type": "boolean"
        },
        "default_client_uri": {
          "format": "uri",
          "type": "string"
        },
        "default_post_logout_redirect_uri": {
          "format": "uri",
          "type": "string"
        },
        "default_redirect_uri": {
          "format": "uri",
          "type": "string"
        },
        "phone_input": {
          "$ref": "#/$defs/PhoneInputConfig"
        },
        "watermark_disabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "UserProfileAttributesAccessControl": {
      "additionalProperties": false,
      "enum": [
        {
          "bearer": "hidden",
          "end_user": "hidden",
          "portal_ui": "hidden"
        },
        {
          "bearer": "hidden",
          "end_user": "hidden",
          "portal_ui": "readonly"
        },
        {
          "bearer": "hidden",
          "end_user": "hidden",
          "portal_ui": "readwrite"
        },
        {
          "bearer": "readonly",
          "end_user": "hidden",
          "portal_ui": "readonly"
        },
        {
          "bearer": "readonly",
          "end_user": "hidden",
          "portal_ui": "readwrite"
        },
        {
          "bearer": "readonly",
          "end_user": "readonly",
          "portal_ui": "readonly"
        },
        {
          "bearer": "readonly",
          "end_user": "readonly",
          "portal_ui": "readwrite"
        },
        {
          "bearer": "readonly",
          "end_user": "readwrite",
          "portal_ui": "readwrite"
        }
      ],
      "properties": {
        "bearer": {
          "$ref": "#/$defs/AccessControlLevelString"
        },
        "end_user": {
          "$ref": "#/$defs/AccessControlLevelString"
        },
        "portal_ui": {
          "$ref": "#/$defs/AccessControlLevelString"
        }
      },
      "type": "object"
    },
    "UserProfileConfig": {
      "additionalProperties": false,
      "properties": {
        "custom_attributes": {
          "$ref": "#/$defs/CustomAttributesConfig"
        },
        "standard_attributes": {
          "$ref": "#/$defs/StandardAttributesConfig"
        }
      },
      "type": "object"
    },
    "VerificationClaimConfig": {
      "additionalProperties": false,
      "properties": {
        "enabled": {
          "type": "boolean"
        },
        "required": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "VerificationClaimsConfig": {
      "additionalProperties": false,
      "properties": {
        "email": {
          "$ref": "#/$defs/VerificationClaimConfig"
        },
        "phone_number": {
          "$ref": "#/$defs/VerificationClaimConfig"
        }
      },
      "type": "object"
    },
    "VerificationConfig": {
      "additionalProperties": false,
      "properties": {
        "claims": {
          "$ref": "#/$defs/VerificationClaimsConfig"
        },
        "code_expiry_seconds": {
          "$ref": "#/$defs/DurationSeconds",
          "minimum": 60
        },
        "criteria": {
          "$ref": "#/$defs/VerificationCriteria"
        }
      },
      "type": "object"
    },
    "VerificationCriteria": {
      "enum": [
        "any",
        "all"
      ],
      "type": "string"
    },
    "WelcomeMessageConfig": {
      "additionalProperties": false,
      "properties": {
        "destination": {
          "$ref": "#/$defs/WelcomeMessageDestination"
        },
        "enabled": {
          "type": "boolean"
        }
      },
      "type": "object"
    },
    "WelcomeMessageDestination": {
      "enum": [
        "first",
        "all"
      ],
      "type": "string"
    }
  },
  "$ref": "#/$defs/AppConfig"
}
```

## Annotated example

```yaml
# The ID of this instance of Authgear.
id: myapp
# Configure different identity behavior.
identity:
  # Configure Login ID Identity.
  login_id:
    # Defines the set of accepted login IDs.
    # By default the user can have
    #
    # At most 1 email
    #
    # If you do not want the defaults, define keys yourselves.
    keys:
      # Define the type of login ID.
      # Valid values are "email" "phone" and "username"
    - type: email
      # How long login ID can be
      # Default is 40.
      max_length: 40
      # Whether the login ID can be modified by the end-user in the settings page.
      modify_disabled: false
    - type: phone
    - type: username
    # Configure the characteristics of some login IDs.
    types:
      # Configure Email Login ID Identity.
      email:
        # Whether + sign should be disallowed in the local part. Default is false.
        block_plus_sign: false
        # Whether the email should be treated case sensitively. Default is false.
        case_sensitive: false
        # Whether . sign should be ignored. Default is false.
        ignore_dot_sign: false
        # Whether email domain blocklist is enabled. Default is false.
        # In addition to setting it to true, you also need to provide
        # `email_domain_blocklist.txt` next to `authgear.yaml`.
        # It is a plaintext file with one domain per line.
        # domain_blocklist_enabled and domain_allowlist_enabled is mutually exclusive.
        domain_blocklist_enabled: false
        # Whether email domain allowlist is enabled. Default is false.
        # In addition to setting it to true, you also need to provide
        # `email_domain_allowlist.txt` next to `authgear.yaml`.
        # It is a plaintext file with one domain per line.
        # domain_blocklist_enabled and domain_allowlist_enabled is mutually exclusive.
        domain_allowlist_enabled: false
        # Whether free email provider domains are blocked. Default is false.
        # It is an auxiliary option of domain_blocklist_enabled so
        # domain_blocklist_enabled must be true for this to take effect.
        block_free_email_provider_domains: false
      # Configure Username Login ID Identity.
      username:
        # Whether the username can only contain `-a-zA-Z0-9_.`. Default is true.
        ascii_only: true
        # Whether reserved usernames are blocked. Default is true.
        block_reserved_usernames: true
        # Whether the username should be treated case sensitively. Default is false.
        case_sensitive: false
        # Whether exclude keyword list is enabled. Default is false.
        # In addition to setting it to true, you also need to provide
        # `reserved_name.txt` next to `authgear.yaml`.
        # It is a plaintext file with one keyword per line.
        exclude_keywords_enabled: false
  # Configure OAuth Identity.
  oauth:
    # Configure external OAuth identity providers.
    providers:
    # Denote the type of the identity provider.
    - type: google
      # alias by default is the same as the value of the type.
      alias: google
      # Client ID and client secret are the credentials you obtain from the specific provider.
      # Please refer to the documentation of the provider.
      # You must separately provide the client secret in the secret config file.
      client_id: google_client_id
      # Configure the verification on the claims derived from the user profile received from the provider.
      claims:
        # Configure the claim "email"
        email:
          # Whether the claim is assumed to be verified.
          # Default is true.
          # That is, by default, all "email" claims from every provider are trusted.
          assume_verified: true
    - type: apple
      alias: apple
      # The client ID for Apple is the services ID.
      # The client secret for Apple is the PEM format of the private key.
      client_id: apple_services_id
      # The key ID of the private key.
      key_id: key_id
      # The team ID of your Apple Developer Account.
      team_id: team_id
    - type: azureadv2
      alias: azure
      client_id: client_id
      # Tenant is either the special value "common", the special value "organizations" or
      # the ID of a Azure AD tenant.
      #
      # Note that when you create the client in Azure Portal,
      # you have to choose which tenant the client intends to interact with.
      #
      # If you wish to allow any microsoft accounts such as
      #
      # - hotmail
      # - Xbox
      # - Outlook
      #
      # to login, then the value must be "common".
      #
      # If you wish to allow any user in any Azure ADs in your Azure account,
      # then the value must be "organizations".
      #
      # Otherwise the value must be the ID of a Azure AD tenant.
      # In this case, only user in that Azure AD can login.
      tenant: common
    - type: wechat
      alias: wechat
      client_id: client_id
      # Wether the given client is bound to an sandbox account.
      # Default is false.
      is_sandbox_account: false
      # The wechat account ID.
      # Wechat account always starts with `gh_`.
      account_id: "gh_foobar"
      # The application type. Valid values are "mobile" and "web".
      app_type: "mobile"
  on_conflict:
    # Configure the behavior in anonymous user promotion when the claimed identity
    # conflicts with an existing identity.
    #
    # Valid values are "error" and "login".
    # Default is "error".
    #
    # For example, the user initially signed up as "user@example.com".
    # Later on the user uninstalled the mobile app.
    # The user installed the mobile app again and forgot they had signed up before.
    # The user continued as anonymous user.
    # The user finally opted to sign up with "user@example.com".
    # At this point, the user has 2 accounts.
    #
    # If the value is "error", an error is shown telling the user that
    # the identity they are claiming has been claimed by another user.
    #
    # If the value is "login", the anonymous user is discarded.
    # And the user simply authenticates themselves as the original user.
    # It is up to the developer to handle account merging.
    promotion: "error"
  # Configure biometric identity
  biometric:
    # Configure whether biometric identity is shown in the settings page.
    # If it is true, the user can view the list of biometric identity in the settings page.
    # They can remove any biometric identity that they will never use again.
    # Default is false.
    list_enabled: false
# Configure different authenticator behavior.
authenticator:
  # Configure OOB-OTP Authenticator.
  oob_otp:
    email:
      # the maximum number of the authenticator the user can have.
      # default is 1.
      maximum: 1
    sms:
      # the maximum number of the authenticator the user can have.
      # default is 1.
      maximum: 1
  # Configure Password Authenticator
  password:
    # Configure password policy
    # All policies are turned off by default.
    policy:
      # Set the minimum length of new password.
      min_length: 10
      # Require new password to have at least 1 digit.
      digit_required: true
      # Require new password to have at least 1 lowercase ASCII character.
      lowercase_required: true
      # Require new password to have at least 1 uppercase ASCII character.
      uppercase_required: true
      # Require new password to have at least 1 symbol character.
      symbol_required: true
      # Disallow password containing the given keywords.
      excluded_keywords:
      - secret
      - admin
      - password
      # Require strong password.
      # The strength of the password is calculated with https://github.com/dropbox/zxcvbn
      # 1 is the weakest level and 5 is the strongest level.
      minimum_guessable_level: 5
      # Determine how long password history is kept.
      history_days: 90
      # Determine how many password history is kept.
      history_size: 10
  # Configure TOTP Authenticator
  totp:
    # the maximum number of the authenticator the user can have.
    # default is 1.
    maximum: 1
# Configure the authentication behavior.
authentication:
  # Determine which identities are enabled.
  #
  # To enable anonymous user, add "anonymous".
  # To enable biometric authentication, add "biometric".
  #
  # By default "login_id" and "oauth" are enabled.
  identities:
  - login_id
  - oauth
  - anonymous
  - biometric
  # Determine which authenticators can be used as primary authenticator.
  # By default only "password" is enabled.
  primary_authenticators:
  - password
  # Determine which authenticators can be used as secondary authenticator.
  # By default only "totp" is enabled.
  secondary_authenticators:
  - totp
  # Configure the MFA behavior.
  #
  # if_exists: The user can add secondary authenticators.
  # If the user has at least one secondary authenticator, then MFA must be performed.
  #
  # required: The user must add secondary authenticators.
  # The user must perform MFA during authentication.
  #
  # disabled: The user cannot add any secondary authenticators.
  #
  # Default is "if_exists"
  secondary_authentication_mode: if_exists
  # Configure Device Token.
  # Device token can be generated during MFA.
  # It is used to skip MFA on the device for future authentication.
  device_token:
    # Determine how long the device token is valid.
    expire_in_days: 30
  # Whether user must be created by admin. Default is false.
  public_signup_disabled: false
  # Configure Recovery Code
  recovery_code:
    # The number of recovery codes. Default is 16.
    count: 16
    # Whether the user can list the recovery codes again. Default is false.
    list_enabled: false
# Configure forgot password behavior
forgot_password:
  # Which forgot password is enabled.
  # The default is true.
  enabled: true
  # How long the reset code remains valid. The default is 1200. That is 20 minutes.
  reset_code_expiry_seconds: 1200
# Configure webhook
hook:
  # How long a single handler can proceed the webhook event before timeout.
  # Default is 5.
  sync_hook_timeout_seconds: 5
  # How long all handlers can proceed the webhook event before timeout.
  # Default is 10.
  sync_hook_total_timeout_seconds: 10
  # Define the list of webhook handlers
  handlers:
    # The event name.
  - event: before_user_create
    # The endpoint of the webhook handler.
    url: https://api.example.com/hook/before_user_create
http:
  # The allowed origin for the HTTP header access-control-allow-origin
  # Default is empty list.
  allowed_origins:
  - https://trusted-third-party-server.com
  # Explicitly set the cookie domain. Default is eTLD+1.
  cookie_domain: https://accounts.myapp.com
  # The expected origin
  # It is used to render an absolute URL in templates.
  public_origin: https://accounts.myapp.com
  # Set the prefix of the cookies written by Authgear in case
  # you have cookie name conflicts you want to avoid.
  # The default prefix is an empty string.
  cookie_prefix: "my_app_"
# Configure default messaging configuration.
messaging:
  # Configure which SMS provider to use.
  # Valid values are "twilio" and "nexmo".
  # You must provide the credentials in secret config.
  sms_provider: "twilio"
# Configure user verification
verification:
  # Determine the verification status criteria.
  # any: User is verified if any of the verifiable claims is verified
  # all: User is verified if all of the verifiable claims are verified
  # Default to any.
  criteria: any
  # Lifetime of verification code, default to 3600 (1 hour)
  code_expiry_seconds: 3600
  # Configure which claims are verifiable and are required to be verified.
  claims:
    # Configure the claim "email"
    email:
      # Whether this claim is verifiable.
      # Default is true.
      enabled: true
      # Whether this claim is required to be verified.
      # Default is true.
      required: true
    # Configure the claim "phone_number"
    phone_number:
      enabled: true
      required: true
# Configure localization.
localization:
  # The list of supported languages.
  # Default is an singleton array of fallback_language.
  supported_languages: ["en"]
  # The fallback language when none of the supported languages match the preferred languages.
  # Default is en.
  fallback_language: en
# Configure OAuth.
oauth:
  # Define the list of known OAuth 2 clients.
  clients:
    # The OAuth 2 client ID.
    # A reasonably long secure random string is recommended.
  - client_id: client_id
    # Whether the client is considered as first party.
    # Default is true.
    # Anonymous user and biometric authentication requires first party client,
    # so normally you should leave it as true.
    is_first_party: true
    # Define a list of allowed redirect URIs.
    # According to OAuth 2 spec, exact match is used.
    # So "http://example.com" does not match "http://example.com/"
    redirect_uris:
    - "com.myapp://host/path"
    # Which grant types are allowed in the token endpoint.
    # "authorization_code" and "refresh_token" must be included for OAuth 2 authorization code flow.
    grant_types:
    - authorization_code
    - refresh_token
    # What response the authorization endpoint can return.
    # "code" must be included for OAuth 2 authorization code flow.
    response_types:
    - code
    # Define a list of allowed post logout redirect URIs.
    # Same as redirect_uris, exact match is used.
    post_logout_redirect_uris:
    - "https://www.myapp.com/post_logout"
    # The lifetime of the access token. Default is 1800 (30 minutes).
    access_token_lifetime_seconds: 1800
    # The lifetime of the refresh token. Default is 31449600 (52 weeks).
    refresh_token_lifetime_seconds: 31449600
    # Whether refresh token idle timeout is enabled. Default is true.
    refresh_token_idle_timeout_enabled: true
    # The idle timeout of the refresh token. Default is 2592000 (30 days).
    refresh_token_idle_timeout_seconds: 2592000
    # Whether the access token is opaque or an JWT.
    # Default is false.
    # Access token as JWT allows you to directly inspect the access token
    # to determine the user, without forwarding the request to the resolver.
    issue_jwt_access_token: false
# Configure session.
session:
  # Whether the cookie is session cookie. Default is false.
  cookie_non_persistent: false
  # Whether the session becomes invalid after idling.
  idle_timeout_enabled: false
  # How long before the session timeout.
  idle_timeout_seconds: 300
  # The lifetime of the session. Default is 86400.
  lifetime_seconds: 86400
ui:
  phone_input:
    # The list of country code to show in the phone number input widget.
    allowlist:
    - 'HK'
    - 'US'
    # The list of country code pinned to the top of the list.
    pinned_list:
    - 'HK'
    # Wether preselection of country code by IP address is disabled.
    # Default is false.
    preselect_by_ip_disabled: false
  # Whether dark theme is disabled.
  # Default is false.
  dark_theme_disabled: false
  # Whether watermark is disabled.
  # Default is false.
  watermark_disabled: false
  # If it is provided, the web UI will have a "Back to App" link.
  default_client_uri: ""
  # If it is provided, the web UI will redirect to the URI after logout.
  default_post_logout_redirect_uri: ""
  # If it is provided, the web UI will redirect to the URI after login.
  default_redirect_uri: ""
# Configure welcome message.
welcome_message:
  # Whether to send the welcome message.
  # Default is false.
  enabled: false
  # Whether to send the welcome message to all addresses or first address.
  # Valid values are first and all.
  # Default is first.
  destination: first
```
