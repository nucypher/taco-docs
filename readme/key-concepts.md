# How TACo works

## Data sharing flow

### 1. Distributed Key Generation&#x20;

<figure><img src="../.gitbook/assets/taco_dkg (2).png" alt=""><figcaption><p>Distributed Key Generation</p></figcaption></figure>

We start from the _adopting developer_'s perspective – i.e. developers of an application that has integrated TACo. Stage 1 – Distributed Key Generation (DKG) – will assign the adopting developer the role of `cohortAuthority`. This grants the developer control over the group of nodes which enforce access control within their app, but no power to decrypt private data shared by their users.  `cohortAuthority` power is easily transferred to a multisig or DAO. &#x20;

Firstly, the `cohortAuthority` samples a group – or cohort – of nodes from the network. Typically a list of nodes to populate a cohort is generated using a replicable random seed, to prove later that the nodes were not hand-picked.&#x20;

The minimum cohort size is 30, and can be as large as 100.&#x20;

Sampled nodes will now conduct a DKG initialization ritual, which involves generating transcripts, aggregating transcripts locally, and cross-verifying the aggregates. If any node submits an incorrect entry, the DKG ritual fails and must start over. That implies that a minimum of one honest party is required at this stage to ensure the secret material is not spoofed.&#x20;

DKG initializations generate private and public material. The public material is used to generate a persistent public key. The cohort holds onto their fragment of private material, which they will later provision to qualifying data consumers.&#x20;

### **2. Encryption**&#x20;

<figure><img src="../.gitbook/assets/taco_encryption.png" alt=""><figcaption><p>Encryption with Conditions</p></figcaption></figure>

Switching to the _data producer_'s perspective – we want to encrypt and share some data. We first specify the conditions for accessing the data.

For example, imagine the data producer is a creator on a decentralized Twitch, and wishes to create a paywall for a special livestream. They will only allow a viewer to decrypt the stream if they (a) hold a minimum number of a special purpose NFT, (b) they need to haves shared a previous stream-unlocking NFT with a friend, and (c) the stream will be non-decryptable by anyone after 24h. \
\
All these conditions are composed into a `conditionSet`and embedded with the ciphertext, which is then delivered to recipients via a transport layer and/or uploaded to a storage layer.&#x20;

### 3. Decryption&#x20;

<figure><img src="../.gitbook/assets/cbd_decryption.png" alt=""><figcaption><p>Condition Fulfillment Decryption</p></figcaption></figure>

Finally, switching to the _data consumer_'s perspective – their first step is to by retrieving the encrypted payload from storage or a transport layer.

Next, the data consumer presents the payload to the cohort, along with whatever authentication message or proof is required to prove their identity. For example, the message can be as simple as a Sign In With Ethereum 'pass through', where the app has already authenticated the user.   \
\
Then, each individual node verifies that the conditions are fulfilled by the data consumer – in our decentralized Twitch streamer example, this would involve retrieving on-chain state to check transaction history, NFT ownership, and blocktime. \
\
If the threshold of nodes (e.e. 26-of-50) validates that the conditions for access are satisfied, then the private material – decryption fragments – are sent back to the data consumer. These are assembled  locally and used to decrypt the payload.&#x20;

Normally, the payload is a sym key that then decrypts the underlying data, via a KEM/DEM mechanism.  

## Key concepts

### **Threshold Decryption**

Under the hood, TACo involves splitting a joint secret – a decryption key – into multiples _shares_ and distributing those among authorized and collateralized node operators (stakers in the Threshold network). A minimum number – a _threshold_ – of those nodes holding the key shares must be online and actively participate in partial decryptions. These are subsequently combined on the requester's client to reconstruct the original plaintext data.

### **Conditionality**

Conditions are 'attached' on a per-ciphertext basis. In other words, each and every payload, message or bit can be access-restricted by a unique set of specified conditions. A range of access condition types can be defined by the _adopting developer_ and/or _data producer_:&#x20;

* EVM-based\
  &#xNAN;_&#x65;.g. Does the requester own a given NFT?_
* RPC-driven\
  &#xNAN;_&#x65;.g. Does the requester have at least X amount of a given token in their wallet?_
* Time-based\
  &#xNAN;_&#x65;.g. Has a predefined period elapsed, after which requests will be ignored?_

Conditions are composable and can be combined in any logical sequence or decision tree. For more on condition logic, check out the [Access Control](../conditions/) section.&#x20;

### **Network parameterization**

In future versions, adopting developers (`cohortAuthority`) will have the option to tweak certain network-level parameters, which affect the collusion-resistance, redundancy, latency and costs of using TACo. These parameters mostly pertain to each cohort of nodes tasked with key material management and condition verification:&#x20;

* The number of decrypting shares _n_\
  i.e. the size of the cohort&#x20;
* The frequency and/or business logic for replacing members of the cohort
* The 'hand-chosen' node address that will always feature in the cohort
* The sampling mechanism for selecting cohort members

This optionality can also be surfaced for end-users – for example, in the form of 'packages' that combine network-level parameters. End-users might choose between a few discrete options, based on their trust, risk and cost preferences.
