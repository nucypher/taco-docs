# OAuth

OAuth (Open Authorization) is an open standard for access delegation, enabling third-party services to access scoped user data without exposing user credentials. Instead of using resource owner credentials to access protected resources from services such as Google, Facebook etc., access tokens with limited capabilities for a set duration are issued to third-party clients by an authorization server with the approval of the resource owner.  These access tokens can be revoked at any time.

A typical OAuth workflow involves:

1. _Authorization Request:_ the application/client requests authorization from the resource owner (user).
2. _Authorization Grant:_ the user gives consent, and the authorization server (e.g. Google, Facebook) issues an authorization grant.
3. _Access Token Request:_ the client exchanges the authorization grant for an access token with the authorization server
4. _Resource Access:_ the client uses the access token to request data from the resource server

```
     +--------+                               +---------------+
     |        |--(1)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(2)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(3)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(3)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(4)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(4)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

## Potential Application to TACo

TACo could function as part of a client system, using an already obtained access token to verify off-chain conditions. A possible use case for OAuth within TACo would involve scenarios where off-chain API endpoints, such as those used for [jsonapicondition.md](json-endpoint-conditions/jsonapicondition.md "mention"), require user authentication via access tokens.

In TACo's case, steps 1-3 of the OAuth process would be handled by the underlying application, not TACo itself. The application simply provides TACo with the access token, which could then be used to verify off-chain conditions requiring user authentication.

We are actively seeking potential use cases and design proposals for implementing OAuth within TACo.

{% hint style="warning" %}
Applications that use OAuth will ultimately rely on centralized user authentication, even though TACo operates in a decentralized manner.
{% endhint %}

## Development References

* [https://github.com/nucypher/nucypher/issues/3559](https://github.com/nucypher/nucypher/issues/3559)
