# What is TACo?

**TACo** (**T**hreshold **A**ccess **Co**ntrol) enables end-to-end encrypted data sharing and communication. There is no requirement to trust a centralized authority or entity, who might unilaterally deny service or even (surreptitiously) decrypt private end-user data.&#x20;

It is currently the only access control layer available to Web3 developers that has been **decentralized from genesis** – via the live, well-collateralized, and battle-tested Threshold network. This means access to all data encrypted using a TACo Mainnet DKG public key is managed by a distributed set of machines, from the very first byte.

{% hint style="warning" %}
All third-party services – digital, Web3 and beyond – impose a trust burden on the user. TACo minimizes this trust burden by disassembling and distributing sensitive cryptographic operations – key generation, access condition verification, and decryption fragment provision – across a large group of independently operated nodes. To understand TACo's trust model and read the _Mainnet_ _Trust Disclosure_, head to the [Trust Assumptions](trust-assumptions/) section.&#x20;
{% endhint %}

TACo provides an **encrypt/decrypt API** for application developers who require:

* end-to-end encrypted channels for their users to share private data safely&#x20;
* programmable, composable access logic and conditionality  &#x20;
* an alternative to centralized access control and its associated trust burdens
* 100% open source software, running across a permissionless network, fully decentralized from day one&#x20;

{% hint style="info" %}
Integrating the TACo layer into your web/web3 app, infrastructure or protocol is straightforward and can be tested in just a few steps – see [Quickstart](quickstart-testnet.md) guide or [Ecoystem Integrations](integrations/).&#x20;
{% endhint %}

TACo's production (mainnet) API is already being utilized in the wild – enforcing decryption rights for media libraries, securing the passage of inheritance assets, and enabling safe account recovery. For more real-world examples, check out [Use Cases](use-cases/).

## Sharing flow overview

Apps that have integrated the TACo SDK automatically interact with the [taco-web](https://github.com/nucypher/taco-web) encrypt/decrypt API. This connects end-users directly to TACo's network of distributed nodes – with no intermediaries.&#x20;

In a typical flow, private data is encrypted client-side by a _data producer._ This data payload will remain encrypted until it reaches the device of a qualifying _data consumer_. From a privacy point of view, this is equivalent to the end-to-end encryption guarantees in a messaging app like Signal. However, TACo is general-purpose and can protect data flows in all types of applications, rather than being optimized for point-to-point textual communication.&#x20;

Whether or not a _data consumer_ qualifies to decrypt and view a given data payload depends on whether they fulfill certain access conditions. These conditions are specified in advance by the producer or owner of that data, or programmed into the application logic on their behalf. For example, a journalist-facing app might predicate access to submitted evidence based on the proven location of the witness (out of harm's way). \
\
To access the data, a given _data consumer_ will have to (1) authenticate themselves and (2) present proof they fulfill the pre-specified conditions. Both are evaluated by a group of TACo nodes, each of which individually validates the data consumer's request by comparing it to retrieved web/web3 state. For example, if perishable health data should not be shared after a certain date, TACo nodes will simply read the UNIX epoch via Ethereum's `block.timestamp` value.&#x20;

If a sufficient number (a 'threshold') of nodes confirm that the requesting _data consumer_ qualifies to see the data, they will send their device some key fragments. These fragments can be put together by the requestor client-side, which produces a decrypting key. This decrypting key can then be used to decrypt the original private data – or more often, a sym key which provides a lightweight conduit to the underlying payload.

## **TACo vs. alternatives**

If your app handles any form of private or sensitive data, your choices as a developer, excluding TACo, are fairly limited:&#x20;

1. **Basic** **Public Key Infrastructure**. Although PKI/PKE typically takes place in the client, and is therefore privacy-preserving (and free), it requires knowing the identifier of a data consumer in advance, the data producer being online at the moment of sharing, and doesn't typically scale beyond a demo or app prototype.&#x20;
2. **Cloud Key Management Systems**. Although straightforward to integrate (and cheap), using a KMS effectively trusts the cloud provider with all of your users' data, given that they ultimately control the master key and can theoretically (and silently) access everything.&#x20;
3. **Decentralized-In-Name-Only Protocols**. Although DINO projects claim they are decentralized (or will eventually decentralize), their 'network' is often a permissioned group of insiders – who all know each other and have zero (crypto)economic incentive not to collude – they can get together to decrypt sensitive data quietly and unbeknownst to end-users. In addition, from a regulatory perspective, a permissioned coterie primarily based in a single jurisdiction is a regulatory single-point-of-failure.&#x20;
4. **Trusted Execution Environments**. Although TEEs have powerful capabilities, they come with an opaque and unpredictable supply chain risk, with many examples of vulnerabilities and exploits. Moreover, they are costly and challenging to run, and therefore difficult to decentralize and expensive – this is restrictive overkill for a lightweight access control service.
