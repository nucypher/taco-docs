# Mainnet Access

{% hint style="info" %}
Developers should familiarize themselves with TACo's [Testnet](../for-developers/get-started-with-tac.md) (Tapir) before requesting access to Mainnet TACo.
{% endhint %}

In order to use TACo in production – i.e. the fully decentralized version of TACo, with independent nodes managing private material – adopting developers must be furnished with their own cohort of nodes which can subsequently be used by the application for encryption/decryption and signing.

The cohort along with the authority of the cohort will be exclusively controlled by the developer until they decide to devolve that power to a DAO, multisig, or any kind of contract. \
\
Developers interested in their own cohort must follow these steps:&#x20;

(1) Make a request fin the TACo discord [server](https://discord.com/channels/411401661714792449/1344417143659171991), which will be answered by TACo's core development team.&#x20;

{% hint style="info" %}
Once a cohort is formed, TACo's core dev team has no power to censor, block or control it in any way.&#x20;
{% endhint %}

(2) Provide the Polygon wallet address you wish to designate as `cohortAdmin`. This address will exclusively control the cohort of nodes.\
\
(3) Prepare the funds (in **DAI**) to pay mainnet usage fees. A single payment is made upfront, before the cohort is formed, and covers both parts of the fee model (duration & per-user). The total sum in DAI depends on:\
a. the number of nodes required (min. 30)\
b. the duration of node rental required (min. 12 months)\
c. _(\*only for decryption)_ the number of unique/reusable encryptors credits required (min. 100)

\
For more details, see the [Mainnet Fees](../mainnet-fees.md) section. \
\
(4) Complete the payment into a proxy contract, which will escrow the funds until the cohort formation is complete. Once completed, the payment will be transferred.&#x20;

{% hint style="info" %}
The proxy contract address will be provided once the request for a Mainnet cohort is accepted.&#x20;
{% endhint %}

{% hint style="warning" %}
In TACo's genesis era, the proxy contract will act as an temporary`sponsor`of the cohort. Note that the `sponsor` role **does not** have any power over the cohort. Rather, the designated Polygon address – the`cohortAdmin`role – will have complete control.
{% endhint %}

(4) Complete the set-up by following the instructions in the TACo [deploying-to-mainnet.md](deploying-to-mainnet.md "mention").
