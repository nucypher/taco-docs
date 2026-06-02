# Value Propositions

TACo is a programmable control plug-in that makes your Web3 app more secure, more private, and much more decentralized. TACo provides APIs for application developers who require:

* end-to-end encrypted channels for their users to perform operations safely
* programmable, composable logic and conditionality
* an alternative to centralized control and its associated trust burdens
* 100% open source software, running across a permissionless network, fully decentralized from day one

**End-to-end encryption for almost everything**\
Built on the privacy-for-everyone principles of mainstream end-to-end encrypted messengers, but useful across a wider set of domains and use cases.&#x20;

**Trust-minimization via threshold cryptography and a collusion-resistant node array**\
Key material management and condition verification are operationally distributed across a diverse array of machines/servers, run by economically independent individuals and commercial entities.

**Powerful conditionality**\
Operations can be made contingent on the fulfillment of predefined [conditions](../for-developers/conditions/), and those conditions attached to varying granularity – eg. for decryption, a single message, or an entire table, or a petabyte of video footage.

**Flexible condition composability**\
Conditions of all types – NFT-holding, RPC, time-based, known keypairs – can be mixed-and-matched using logical operators and flexible prefix notation into virtually any desired combination. Conditions can also be flexibly surfaced at different stages of runtime.

**Full control over access managers & network parameters**\
Developers have total and perpetual control over the cohort(s) of nodes which manage allowed application operations. Network parameters, such as the size or composition of the cohort, can be tuned directly by the developer, or packaged into user-facing optionality to accommodate diverse risk preferences.

#### **TACo vs. alternatives**

If your app handles any form of private or sensitive data, or require automated signing workflows your choices as a developer, excluding TACo, are fairly limited:

1. **Basic** **Public Key Infrastructure**. Although PKI/PKE typically takes place in the client, and is therefore privacy-preserving (and free), it requires knowing the identifier of a data consumer in advance, the data producer being online at the moment of sharing, and doesn't typically scale beyond a demo or app prototype.
2. **Cloud Key Management Systems**. Although straightforward to integrate (and cheap), using a KMS effectively trusts the cloud provider with all of your users' data, given that they ultimately control the master key and can theoretically (and silently) access everything.
3. **Decentralized-In-Name-Only Protocols**. Although DINO projects claim they are decentralized (or will eventually decentralize), their 'network' is often a permissioned group of insiders – who all know each other and have zero (crypto)economic incentive not to collude – they can get together to decrypt sensitive data quietly and unbeknownst to end-users. In addition, from a regulatory perspective, a permissioned coterie primarily based in a single jurisdiction is a regulatory single-point-of-failure.
4. **Trusted Execution Environments**. Although TEEs have powerful capabilities, they come with an opaque and unpredictable supply chain risk, with many examples of vulnerabilities and exploits. Moreover, they are costly and challenging to run, and therefore difficult to decentralize and expensive – this is restrictive overkill for a lightweight access control service.
5. **Secret Sharing Schemes (SSS).** Tends to be best suited of one-time recovery scenarios since the secret is explicitly revealed after combining secret shares. This contrasts with threshold decryption systems like TACo, where partial decryption shares are combined for ciphertext decryption without ever exposing the underlying private key.



##

<br>
