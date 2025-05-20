# Value Propositions

TACo provides an **encrypt/decrypt API** for application developers who require:

* end-to-end encrypted channels for their users to share private data safely&#x20;
* programmable, composable access logic and conditionality  &#x20;
* an alternative to centralized access control and its associated trust burdens
* 100% open source software, running across a permissionless network, fully decentralized from day one&#x20;

### TACo is a programmable access control plug-in that makes your Web3 app more secure, more private, and much more decentralized.&#x20;

**End-to-end encryption for everything**\
Built on the privacy-for-everyone principles of mainstream end-to-end encrypted messengers, but useful across a wider set of domains and use cases. Everything from live-streaming to connected vehicles, from secret recovery to decentralized AI compute, from journalist protection to reproductive health care.&#x20;

**Trust-minimization via threshold cryptography and a collusion-resistant node array**\
Key material management and access condition verification are operationally distributed across a diverse array of machines/servers, run by economically independent individuals and commercial entities.&#x20;

**Powerful, per-ciphertext conditionality**\
Future access to data can be made contingent on the fulfillment of predefined conditions, and those conditions attached to any granularity of data payload – a single message, an entire table, or a petabyte of video footage.&#x20;

**Flexible condition composability**\
Conditions of all types – NFT-holding, RPC, time-based, known keypairs – can be mixed-and-matched using logical operators and flexible prefix notation into virtually any desired combination. Conditions can also be flexibly surfaced at different stages of runtime.&#x20;

**Full control over access managers & network parameters**\
Developers have total and perpetual control over the cohort(s) of nodes which manage access to a given data payload, user segment or entire application. Network parameters, such as the size or composition of the cohort, can be tuned directly by the developer, or packaged into user-facing optionality to accommodate diverse risk preferences.\
&#xNAN;_&#x4E;ote: parameter tuning will be available in future versions of TACo._&#x20;

**Threshold's track record for uptime, reliability and competence**\
The Threshold network's multi-app model incentivizes all TACo nodes to simultaneously provision service to tBTC, and abide by its strict availability requirements. With over [$130m](https://dune.com/threshold/tbtc) in TVL protected by Threshold nodes, TACo 'piggybacks' on tBTC's uptime, reliability, technical competence, and security track record.&#x20;

**Keypair-only decryption via** [**PRE Extension**](broken-reference)\
If even stricter security guarantees are required, and data recipients' public keys are known in advance, developers may opt for end-user data to be re-encrypted by node operators such that they are only decryptable by pre-designated clients or public keys.\
&#xNAN;_&#x4E;ote: PRE available on request._&#x20;

## **TACo vs. alternatives**

If your app handles any form of private or sensitive data, your choices as a developer, excluding TACo, are fairly limited:&#x20;

1. **Basic** **Public Key Infrastructure**. Although PKI/PKE typically takes place in the client, and is therefore privacy-preserving (and free), it requires knowing the identifier of a data consumer in advance, the data producer being online at the moment of sharing, and doesn't typically scale beyond a demo or app prototype.&#x20;
2. **Cloud Key Management Systems**. Although straightforward to integrate (and cheap), using a KMS effectively trusts the cloud provider with all of your users' data, given that they ultimately control the master key and can theoretically (and silently) access everything.&#x20;
3. **Decentralized-In-Name-Only Protocols**. Although DINO projects claim they are decentralized (or will eventually decentralize), their 'network' is often a permissioned group of insiders – who all know each other and have zero (crypto)economic incentive not to collude – they can get together to decrypt sensitive data quietly and unbeknownst to end-users. In addition, from a regulatory perspective, a permissioned coterie primarily based in a single jurisdiction is a regulatory single-point-of-failure.&#x20;
4. **Trusted Execution Environments**. Although TEEs have powerful capabilities, they come with an opaque and unpredictable supply chain risk, with many examples of vulnerabilities and exploits. Moreover, they are costly and challenging to run, and therefore difficult to decentralize and expensive – this is restrictive overkill for a lightweight access control service.
5. **Secret Sharing Schemes (SSS).** Tends to be best suited of one-time recovery scenarios since the secret is explicitly revealed after combining secret shares. This contrasts with threshold decryption systems like TACo, where partial decryption shares are combined for ciphertext decryption without ever exposing the underlying private key.
