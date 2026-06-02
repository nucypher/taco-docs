# Fee Model

{% hint style="warning" %}
Threshold Access Control is not currently supported by an stable cohort of node operators running TACo clients, and hence there is no active fee model or paywall. The network will be relaunched by WEDF in Q3 2026.

Until then, this page serves solely as a open source reference and blueprint for the community.&#x20;
{% endhint %}

### `sponsor` & `cohortAdmin` Roles

Adopting developers pay in advance for use of TACo mainnet by transacting with the relevant contract, triggering the cohort formation to have a group of TACo nodes under their exclusive control. There are two key roles associated with a given DKG ritual:

* `sponsor`_:_ This address sends the initial transaction, triggering the cohort formation, and also paying the upfront fees. This address does not have any special privileges or power over the cohort of nodes. Indeed, any EOA can create a new cohort or sponsor an existing cohort.
* `cohortAdmin`_:_ This address has unilateral control over the parameters governing the cohort of nodes. Note that the `cohortAdmin` address does not have to participate during the initiation process.

The `sponsor` and `cohortAdmin` roles can use the same address or use different addresses. For example, the `cohortAdmin` could be a cold wallet address, while the `sponsor` might simply be a one-off software address. External developers may also prefer to set a DAO, a Multisig, or any kind of smart contract as the `cohortAdmin`, which would reduce the trust burden on their end-users with respect to control over encryptors and the TACo cohort.

Currently, cohort formation is not permissionless and must be pre-approved in the [`Coordinator`](https://github.com/nucypher/nucypher-contracts/blob/main/contracts/contracts/coordination/Coordinator.sol) contract by the NuCypher team.&#x20;

### Fee structure

#### **Threshold Decryption**

Adopting developers pay for the TACo service via a dual fee model, which covers:

1. Availability of the service, via a **duration-based fee**\
   \&#xNAN;_Currently 0.75 DAI per node per day_
2. Usage of the service via a **fee based on the number of unique data producer identities encrypting data at any one time**\
   \&#xNAN;_Currently 2.5 DAI per encryptor slot per year_\
   \
   Note that encrypting privileges can be added and removed from identities/addresses at will, without charge or limit, provided the sponsor has pre-paid for sufficient credits and there are encryptor slots available.

There is no charge, payment gate or limit on:

* encryptions
* types or combinations of conditions specified
* unique requestor identities
* throughput/number of requests
* decryptions
* condition validations/invalidations
* additions/removals of addresses to/from the `authAdmin` list
* additions/removals of addresses to/from the encryptor allowlist
* any other communication with the network or API

