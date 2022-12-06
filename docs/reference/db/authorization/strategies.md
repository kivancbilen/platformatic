# Strategies

<!-- TODO: Update diagrams with the updated versions I created for my API adventure talk -->

## Introduction

Platformatic DB supports the following authorization strategies:

- [JSON Web Token (JWT)](#json-web-token-jwt) with support for [JSON Web Key Sets (JWKS)](#json-web-key-sets-jwks)
- [Webhook](#webhook)
- [HTTP headers (development only)](#http-headers-development-only)

<!-- TODO: Clarify/expand this section and put it somewhere else

## JWT and Webhook

It's possible to configure the server to first try to validate a JWT token and if that is not found,
forward the request to the webhook.

-->

## JSON Web Token (JWT)

The [JSON Web Token](https://jwt.io/) (JWT) authorization strategy is built on top of
[fastify-jwt](https://github.com/fastify/fastify-jwt).

![Platformatic DB JWT integration](./images/jwt.png)

To configure it, the quickest way is to pass a shared `secret`, for example:

```json
{
  "authorization": {
    "jwt": {
      "secret": "<shared-secret>"
    }
  }
}
```

For more complex configurations, please check [fastify-jwt options](https://github.com/fastify/fastify-jwt#options).

### JSON Web Key Sets (JWKS)

The JWT authorization strategy includes support for [JSON Web Key](https://www.rfc-editor.org/rfc/rfc7517) Sets.

To configure it:

```json
{
  "authorization": {
    "jwt": {
      "jwks": {
        "allowedDomains": [
          "https://ISSUER_DOMAIN"
        ]
      }
    }
  }
}
```

More [get-jwks options](https://github.com/nearform/get-jwks#options) can be specified.

When a JSON Web Token is included in a request to Platformatic DB, it retrieves the
correct public key from `https:/ISSUER_DOMAIN/.well-known/jwks.json` and uses it to
verify the JWT signature. The token carries all the informations, like the `kid`,
which is the key id used to sign the token itself, so no other configuration is required.

It's also possible to enable JWKS without any options:

```json
{
  "authorization": {
    "jwt": {
      "jwks": true
    }
  }
}
```

In this case, the JWK URL is calculated from the `iss` (issuer) field of JWT, so
every JWT token from an issuer that exposes a valid JWKS token will pass the validation.
For that reason, **this configuration should be used only for development**, while
in every other case the `allowedDomains` should be specified.

### JWT Custom Claim Namespace

JWT claims can be namespaced to avoid name collisions. If so, we will receive tokens
with custom claims such as: `https://platformatic.dev/X-PLATFORMATIC-ROLE`
(where `https://platformatic.cloud/ is the namespace).
If we want to map these claims to user metadata removing our namespace, we can
specify the namespace in the JWT options:

```json
  ...

  "authorization": {
    "jwt": {
      "namespace": "https://platformatic.dev/"
      }
    },
  }

  ...
```

With this configuration, the `https://platformatic.dev/X-PLATFORMATIC-ROLE` claim
is mapped to `X-PLATFORMATIC-ROLE` user metadata.

## Webhook

<!-- TODO: authenticate OR authorize? -->

Platformatic DB can use a webhook to authenticate requests.

![Platformatic DB Webhook integration](./images/webhook.png)

In this case, the URL is configured on authorization:

```json
{
  "authorization": {
    "webhook": {
      "url": "<webhook url>"
    },
  }
}
```

When a request is received, Platformatic sends a `POST` to the webhook, replicating
the same body and headers, except for:

- `host`
- `connection`

In the Webhook case, the HTTP response contains the roles/user information as HTTP headers.

## HTTP headers (development only)

:::danger
Passing an admin API key via HTTP headers is highly insecure and should only be used
during development or within protected networks.
:::

If a request has `X-PLATFORMATIC-ADMIN-SECRET` HTTP header set with a valid `adminSecret`
(see [configuration reference](/reference/db/configuration.md#authorization)) the
role is set automatically as `platformatic-admin`, unless a different role is set for
user impersonation (which is disabled if JWT or Webhook are set, see [below](#user-impersonation)).

![Platformatic DB HTTP Headers](./images/http.png)

<!-- TODO: Unclear what the following paragraph means -->

Also, the following rule is automatically added to every entity, allowing the user
that presented the `adminSecret` to perform any operation on any entity:

```json
{
  "role": "platformatic-admin",
  "find": false,
  "delete": false,
  "save": false
}
```
