# Any (Major) EVM Chain Condition Support

{% hint style="info" %}
This feature is only available on the `lynx`testnet.
{% endhint %}

As the adoption of TACo continues to grow, one challenge that arises is supporting an increasing number of blockchains. Currently, nodes are required to configure a dedicated RPC URL for each blockchain supported for conditions. For example, regardless of the TACo domain (`mainnet`or testnet), nodes must configure an RPC URL for Ethereum (Mainnet/Sepolia). and Polygon (Mainnet/Amoy). Those chains would represent the available chains that can be used for on-chain conditions.

While this configuration ensures reliability, it becomes cumbersome as more blockchains are added to TACo's ecosystem for conditions. Each new blockchain requires nodes to manually set up and manage its RPC URL, increasing the administrative burden over time.

Currently, TACo nodes employ a fallback mechanism to enhance reliability by utilizing a list of public RPC URLs. In this setup, the configured RPC URL is given priority, but if it becomes temporarily unavailable, the node automatically switches to one of the fallback public RPC URLs. For instance, on mainnet, these backup URLs are drawn from the [`mainnet.json` file in the `nucypher/chainlist` repository](https://github.com/nucypher/chainlist/blob/main/mainnet.json) and are specifically limited to Ethereum Mainnet and Polygon Mainnet chain ids.

Expanding on the existing fallback mechanism, we are investigating a more streamlined approach. Rather than requiring nodes to configure specific RPC URLs for each supported blockchain for conditions, we are experimenting with maintaining a curated list of public RPC endpoints for most EVM-compatible chains. This approach allows nodes to utilize these pre-defined public RPC URLs, removing the need for dedicated configurations for every blockchain. Since most chains offer multiple public RPC URLs, the list provides built-in redundancy, ensuring reliability during condition verification.

The list of supported chains on `lynx` is available in the [`lynx.json`file in the `nucypher/chainlist`repository](https://github.com/nucypher/chainlist/blob/main/lynx.json). This comprehensive list enables rapid prototyping for dApps across multiple blockchains using the `lynx`testnet, addressing a growing demand from developers.

That said, there are valid concerns about whether this approach offers sufficient reliability for production-level applications. Consequently, this feature is currently being tested exclusively on the `lynx` testnet.

A potential solution for production could involve integrating API keys from services like Infura, Alchemy, or dRPC. Using these keys, nodes could dynamically derive RPC URLs for various blockchains. However, this method introduces a degree of centralization, as the API key templates would dictate which RPC services TACo supports.

We are actively seeking feedback from adopters to refine our approach and ensure it aligns with their needs. Please chat with us in the #build channel in the [TACo Discord](https://discord.gg/buildwithtaco), or file [GitHub issues](https://github.com/nucypher/nucypher/issues).

## Development References

* [https://github.com/nucypher/nucypher/pull/3569](https://github.com/nucypher/nucypher/pull/3569)
* [https://github.com/nucypher/chainlist/pull/2](https://github.com/nucypher/chainlist/pull/2)
* [https://github.com/nucypher/chainlist/pull/3](https://github.com/nucypher/chainlist/pull/3)
