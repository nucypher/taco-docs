# Access Control

## **TACo vs. alternatives**

If your app handles any form of private or sensitive data, or require automated signing workflows your choices as a developer, excluding TACo, are fairly limited:&#x20;

1. **Basic** **Public Key Infrastructure**. Although PKI/PKE typically takes place in the client, and is therefore privacy-preserving (and free), it requires knowing the identifier of a data consumer in advance, the data producer being online at the moment of sharing, and doesn't typically scale beyond a demo or app prototype.&#x20;
2. **Cloud Key Management Systems**. Although straightforward to integrate (and cheap), using a KMS effectively trusts the cloud provider with all of your users' data, given that they ultimately control the master key and can theoretically (and silently) access everything.&#x20;
3. **Decentralized-In-Name-Only Protocols**. Although DINO projects claim they are decentralized (or will eventually decentralize), their 'network' is often a permissioned group of insiders – who all know each other and have zero (crypto)economic incentive not to collude – they can get together to decrypt sensitive data quietly and unbeknownst to end-users. In addition, from a regulatory perspective, a permissioned coterie primarily based in a single jurisdiction is a regulatory single-point-of-failure.&#x20;
4. **Trusted Execution Environments**. Although TEEs have powerful capabilities, they come with an opaque and unpredictable supply chain risk, with many examples of vulnerabilities and exploits. Moreover, they are costly and challenging to run, and therefore difficult to decentralize and expensive – this is restrictive overkill for a lightweight access control service.
5. **Secret Sharing Schemes (SSS).** Tends to be best suited of one-time recovery scenarios since the secret is explicitly revealed after combining secret shares. This contrasts with threshold decryption systems like TACo, where partial decryption shares are combined for ciphertext decryption without ever exposing the underlying private key.
