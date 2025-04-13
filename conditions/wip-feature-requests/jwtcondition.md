# JWTCondition

The `JWTCondition` validates [JSON Web Tokens (JWTs)](https://datatracker.ietf.org/doc/html/
rfc7519) against a specified public key. It supports standard JWT claims like expiration time and "not before" time. This condition type enables integration with existing Web2 authentication and authorization services.

The JWT standard's flexibility allows for various use cases, including:
- DRM platforms
- Content distribution
- Identity management

## Centralization Considerations

In Web2 environments, JWT issuers are typically centralized. While this isn't a limitation for TACo, it's important to note. In Web3 settings, TACo can work with decentralized JWT issuers that use threshold digital signatures (like threshold ECDSA). From a verification perspective, TACo remains agnostic to the token issuing environment.

## Properties

* `jwtToken`: The JWT token to validate
* `publicKey`: A string containing the digital signature public key in PEM format
* `expectedIssuer` (Optional): A string representing the JWT issuer. If provided, it must match the token's [`issuer` claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1)

**Error Handling**

The condition will fail and access will be denied in the following cases:

* If the JWT token is malformed or cannot be parsed
* If the token's signature cannot be verified with the provided public key
* If the token has expired (when `exp` claim is present)
* If the token is not yet valid (when `nbf` claim is present)
* If the `expectedIssuer` is provided but doesn't match the token's `iss` claim
* If any required claims specified in the condition are missing from the token

## Example

```typescript
import { conditions } from '@nucypher/taco';

const jwtCondition = new conditions.base.jwt.JWTCondition({
  jwtToken: ":authToken", // Context variable for the JWT token
  publicKey: "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...", // Public key in PEM format
  expectedIssuer: "https://some-jwt-issuer.com", // Optional issuer validation
});

// The condition would be satisfied if the JWT token:
// 1. Has a valid signature verifiable with the provided public key
// 2. Has not expired (if exp claim is present)
// 3. Is currently valid (if nbf claim is present)
// 4. Was issued by "https://some-jwt-issuer.com" (if expectedIssuer is provided)
```

## Development References

* Client-side:
  * [https://github.com/nucypher/taco-web/pull/604](https://github.com/nucypher/taco-web/pull/604)
* Server-side:
  * [https://github.com/nucypher/nucypher/pull/3570](https://github.com/nucypher/nucypher/pull/3570)
