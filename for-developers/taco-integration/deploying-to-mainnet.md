# Mainnet Deployment

{% hint style="warning" %}
Threshold Access Control is not currently supported by an stable cohort of node operators running TACo clients, nor the surrounding infrastructure to make mainnet access straightforward (e.g. [Porter](../../reference/architecture/porter.md)). The network will be relaunched by WEDF in Q3 2026.

Until then, this page serves solely as a open source reference and blueprint for the community.
{% endhint %}

There are no substantial changes in the code when one switches from using _testnet_ domain to _mainnet_ domain. However, when calling to the API functions, take into account the following:

* The RPC provider URL (Infura, Alchemy, etc) must be changed to from Polygon Amoy (testnet) to Polygon (mainnet) since the L2 of TACo's mainnet domain is the latter.
* The domain variable must be set to `domains.MAINNET`.
* The ID for the cohort must be set to the relevant cohort.

<pre class="language-typescript"><code class="lang-typescript">import { domains } from "@nucypher/taco"

// This should be a environment variable
const rpcProviderUrl = "https://polygon-mainnet.infura.io/v3/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

<strong>const provider = new ethers.providers.JsonRpcProvider(rpcProviderUrl);
</strong><strong>const domain = domains.MAINNET;
</strong><strong>const id = 0 // Replace by the relevant ID
</strong>
<strong>// TACo operations
</strong><strong>...
</strong></code></pre>
