# Mainnet Deployment

Following a successful prototyping/testing of the integration using TACo's [Testnet(s)](../for-developers/get-started-with-tac.md), we are ready to switch to TACo Mainnet and deploy our application to production. TACo Mainnet is the fully decentralized version of the TACo API and network.&#x20;

## Cohort Formation&#x20;

{% hint style="info" %}
In order to use TACo in production, developers must request [Mainnet Access](mainnet-taco-beta-program.md).
{% endhint %}

## Mainnet use of TACo post-cohort formation

There are no substantial changes in the code when one switches from using _testnet_ domain to _mainnet_ domain. However, when calling to the API functions, take into account the following:

* The RPC provider URL (Infura, Alchemy, etc) must be changed to from Polygon Amoy (testnet) to Polygon (mainnet) since the L2 of TACo's mainnet domain is the latter.
* The domain variable must be set to `domains.MAINNET`.
* The ID for the cohort must be set to the relevant cohort.

<pre class="language-typescript"><code class="lang-typescript">import { domains } from "@nucypher/taco"

// This should be a environment variable
const rpcProviderUrl = "https://polygon-mainnet.infura.io/v3/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
<strong>
</strong><strong>const provider = new ethers.providers.JsonRpcProvider(rpcProviderUrl);
</strong><strong>const domain = domains.MAINNET;
</strong><strong>const id = 0 // Replace by the relevant ID
</strong><strong>
</strong><strong>// TACo operations
</strong><strong>...
</strong></code></pre>

