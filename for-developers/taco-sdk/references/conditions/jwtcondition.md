# JWT Conditions

The `JWTCondition` validates [JSON Web Tokens (JWTs)](https://datatracker.ietf.org/doc/html/rfc7519) against a specified public key. It supports standard JWT claims like expiration time and "not before" time. This condition type enables integration with existing Web2 authentication and authorization services.

The JWT standard's flexibility allows for various use cases, including:

* DRM frameworks and platforms
* Content distribution
* Identity management
* Access-controlled agentic workflows

## Centralization Considerations

In Web2 environments, JWT issuers are typically trusted central authorities. The presence of _centralized issuance_ of JWTs does not impact the _decentralized verification_ of those JWTs by the TACo network, but it does have trust implications for the system as a whole.\
Conversely, in Web3 settings, TACo is fully compatible with decentralized JWT issuers – for example, those that leverage threshold digital signatures like threshold ECDSA. From a verification perspective, TACo remains agnostic to the token issuing environment or entity.

## Properties

* `jwtToken`: The JWT context variable to be instantiated during decryption with a JWT and validated.
* `publicKey`: A string containing the digital signature public key in PEM format
* `expectedIssuer` (Optional): A string representing the JWT issuer. If provided, it must match the token's [`issuer` claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1)

## Error Handling

The condition will fail and access will be denied in the following cases:

* If the JWT is malformed or cannot be parsed
* If the JWT's signature cannot be verified with the provided public key
* If the JWT has expired (when [`exp` (Expiration Time) claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.4) is present)
* If the JWT is not yet valid (when [`nbf` (Not Before) claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.5) is present)
* If the `expectedIssuer` is provided but doesn't match the JWT's [`iss` (Issuer) claim](https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1)
* If any required claims specified in the condition are missing from the JWT

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
// 2. Has not expired (if exp claim is present in the JWT)
// 3. Is currently valid (if nbf claim is present in the JWT)
// 4. Was issued by "https://some-jwt-issuer.com" (since expectedIssuer was specified in the condition)
```

## Development References

* Client-side:
  * [https://github.com/nucypher/taco-web/pull/604](https://github.com/nucypher/taco-web/pull/604)
* Server-side:
  * [https://github.com/nucypher/nucypher/pull/3570](https://github.com/nucypher/nucypher/pull/3570)
