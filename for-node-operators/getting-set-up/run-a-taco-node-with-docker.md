# Run a TACo Node with Docker

{% hint style="warning" %}
TACo is currently in the process of transitioning away from the [Threshold Network](https://threshold.network) to a new staking infrastructure. As a result, **no new TACo stakes or node operators will be permitted until this transition is fully completed.**

For the latest updates and announcements regarding this transition, please stay connected via the [TACo Discord server](https://discord.gg/buildwithtaco).
{% endhint %}

## Before you begin

* Running a TACo node requires maintenance and comes with certain constraints. Please review the duties expected of a node operator, and make sure you are comfortable with the minimum deauthorization delay of 6 months.
* Please review the system requirements for provisioning the TACo service.
* Your operator account will need to be funded with at least 15 POL (Polygon POS) to connect to the Threshold network. You should transfer these funds after getting the node running.
* Once TACo is running smoothly on your machine or VPS, the next step is to authorize your stake to the TACo app and register/bond the node to that provider address.

## Technical Overview

The overall procedure for setting up a TACo Node is as follows:\
\
1\. Get Docker Image\
2\. Create Operator Ethereum Wallet\
3\. Set Passwords\
4\. Initialize the Node\
5\. Launch the Node\
6\. (Optional) Automatic Updates\
7\. (Optional) Expose Prometheus Metrics

This excludes registration and authorization, which you should attempt once completing the steps on this page.

## 1. Get Docker Image

If Docker is not already installed on your server, follow the official Docker installation [instructions](https://docs.docker.com/engine/install/ubuntu/). If you are using a DigitalOcean VPS, you may find these [instructions](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04) helpful.

Pull the latest Docker image from NuCypher's primary repo:

```bash
docker pull nucypher/nucypher:latest
```

Note that NuCypher is a contributing team to the Threshold Network and the primary developers of the TACo application.

## 2. Create Operator Ethereum Wallet

The _operator_ is a dedicated Ethereum wallet address that will be used to identify your TACo node. You will map this address to a staking provider on the threshold dashboard later. This mapping step is also referred to as 'bonding' and 'registering'. This wallet must be in Geth-compatible JSON format ([Web3 Secret Storage Format](https://github.com/ethereum/wiki/wiki/Web3-Secret-Storage-Definition)) and can be generated with a variety of publicly available tools like [go-ethereum](https://geth.ethereum.org/) ("geth") or [MyCryptoWallet](https://mycrypto.com/).

In this step you will create an ethereum software wallet using Geth, following these installation [instructions](https://geth.ethereum.org/docs/getting-started/installing-geth). Note that installing Geth on an Ubuntu server can generate errors with newer versions. To avoid this, choose a long term support version – e.g. Ubuntu 20.04 (LTS).

Once Geth is installed, create a new ethereum wallet:

```bash
geth account new
```

A successful output should look like this:

```bash
$ geth account new
...
Your new account is locked with a password. Please give a password. Do not forget this password.
Password: 
Repeat password: 

Your new key was generated

Public address of the key:   0xdEB634255A534870505D085717898F1A8A0B53d8
Path of the secret key file: /home/user/.ethereum/keystore/UTC--2023-12-08T18-58-13.845048610Z--deb634255a534870505d085717898f1a8a0b53d8
...
```

{% hint style="info" %}
Take note of your new operator address and secret key file path, as you will need them in the next steps.
{% endhint %}

{% hint style="danger" %}
Secure and back-up your password and operator secret key file off-site. Loss of your operator wallet or password may result in service disruption, loss of rewards, and/or manual intervention.
{% endhint %}

## 3. Set Passwords

There are two passwords associated with a TACo node:

* _nucypher keystore password_ - This password is used to encrypt your network participation keys. You can create this password now.
* _operator password_ - This password will be used to unlock you operator ethereum wallet. Enter the same password you used when you created your (geth) wallet.

Create a plain text file named `nucypher.env` containing the following variables. Replace <...> with your passwords.

```bash
NUCYPHER_KEYSTORE_PASSWORD=<YOUR NUCYPHER KEYSTORE PASSWORD>
NUCYPHER_OPERATOR_ETH_PASSWORD=<YOUR OPERATOR ETH ACCOUNT PASSWORD>
```

## 4. Initialize the Node

TACo nodes must be initialized before launching. This is an interactive one-time step that will create network participation keys and an initial JSON configuration file:

{% hint style="warning" %}
Initializing a TACo node is a one-time procedure that requires you to secure a mnemonic seed phrase. This phrase is used to derive cryptographic keys used in TACo protocols. It is critical to maintain custody of the mnemonic in case of password loss or host relocation.
{% endhint %}

{% hint style="danger" %}
Loss of the TACo secret mnemonic may result in your stake being slashed.
{% endhint %}

<pre class="language-bash"><code class="lang-bash">docker run -it --rm                        \
--name ursula-init                         \
<strong>-v ~/.local/share/nucypher:/root/.local/share/nucypher:rw \
</strong>-v ~/.ethereum/:/root/.ethereum:ro         \
-p 9151:9151                               \
--env-file nucypher.env                    \
nucypher/nucypher:latest                   \
<strong>nucypher ursula init                       \
</strong>--signer keystore:///root/.ethereum/keystore/&#x3C;WALLET FILENAME> \
--domain mainnet                           \
<strong>--eth-endpoint &#x3C;ETH PROVIDER URI>          \
</strong><strong>--polygon-endpoint &#x3C;POLYGON PROVIDER URI>  \
</strong>--operator-address &#x3C;OPERATOR ADDRESS>      
</code></pre>

Replace the following values with your own:

* `<ETH ENDPOINT URI>` The URI of an ethereum JSON-RPC endpoint (e.g. `https://infura.io/…`)
* `<POLYGON ENDPOINT URI>` The URI of a polygon JSON-RPC endpoint (e.g.. `https://infura.io/...`)
* `<WALLET FILENAME>` The _filename_ of your operator software wallet
* `<OPERATOR ADDRESS>` The dedicated ethereum address to be used by the TACo node

Follow the in-terminal prompts. You will see a public key for your TACo node and be assigned a mnemonic phrase.

## 5. Launch the Node

{% hint style="danger" %}
The first time a taco node is launched the public key generated in the previous step is committed on-chain.  After this commitment, loss of the private keys is a protocol offensive that will result in reward withholding and/or stake slashing.
{% endhint %}

Run the following command to launch the node:

```bash
docker run -d                     \
--name ursula                     \
--restart unless-stopped          \
-v ~/.local/share/nucypher:/root/.local/share/nucypher:rw \
-v ~/.cache/nucypher:/root/.cache/nucypher:rw \
-v ~/.ethereum/:/root/.ethereum:ro   \
-p 9151:9151                      \
--env-file nucypher.env           \
nucypher/nucypher:latest          \
nucypher ursula run 
```

Successful execution will resemble this example:

```bash
$ docker run -d --name ursula ...
5ecaa04eb319da576c3b2fa2b8aee9cc1a7079cd2675e3202047b50174696a84
```

### View Logs

When your node starts up, it will connect to Polygon and Ethereum mainnet to determine if the two qualification criteria are satisfied:

1\. Operator account is funded with MATIC (Polygon POS); at least 15 MATIC is recommended.\
2\. Operator account is mapped/bonded to a staking provider.

{% hint style="info" %}
Operator bonding must be performed on the Threshold Staking dashboard. Once complete there is a \~20 minute waiting period for your node's status to be automatically bridged to Polygon.\
\
If your node is not bonded and synced the following message will be displayed in logs during this waiting period:\
\
`! Bonded staking provider address 0xDB1970...0991D096 on Mainnet not yet synced to child application on Polygon/Mainnet ; waiting for sync`
{% endhint %}

\
Verify your node is running correctly by viewing the logs:

```bash
docker logs -f ursula
```

The following is an example of the expected output for a TACo node that is both funded with\
POL and correctly bonded to an operator on the threshold dashboard.

```bash
...
! Bonded staking provider address 0xDB1970...0991D096 on Mainnet not yet synced to child application on Polygon/Mainnet ; waiting for sync
✓ Operator 0x27cd20d513cD3aB1D030e60f3aFb75599A33Bc2D is bonded to staking provider 0xDB1970e65B501f906f0fD220164800a0E824456E
! Checking provider's DKG participation public key for 0xDB1970e65B501f906f0fD220164800a0E824456E on Polygon/Mainnet at Coordinator 0xE74259e3dafe30bAA8700238e324b47aC98FE755
Broadcasting SETPROVIDERPUBLICKEY Legacy Transaction (0.021561263955835524 ETH @ 119.649197331 gwei)
TXHASH 0xa034bc89f8f30980e1222c9a17a71683849119ae7953e7c04d659a057f77f384
Waiting 600 seconds for receipt
✓ Successfully published provider's DKG participation public key for 0xDB1970e65B501f906f0fD220164800a0E824456E on Polygon/Mainnet with txhash 0x40cda7a3120d4555e64802e813f2fd9de2ea5c3616cff24393d332daa92ce2d2)
✓ Start Operator Bonded Tracker
✓ Rest Server https://182.16.254.42:9151
Working ~ Keep Ursula Online!
```

{% hint style="success" %}
Working \~ Keep Ursula Online! Indicates successful launch :tada:
{% endhint %}

## 6. (Optional) Automatic Updates

You can optionally configure your server to automatically update any running docker containers using watchtower. This will automatically relaunch your node with the same commands and environment when an update to nucypher is published:

```bash
docker run --detach \
--name watchtower   \
--restart unless-stopped \
--volume /var/run/docker.sock:/var/run/docker.sock \
containrrr/watchtower ursula --cleanup
```

{% hint style="info" %}
This command assumes the name of your node docker container is `ursula`
{% endhint %}

For more information check out the official Watchtower [documentation](https://containrrr.dev/watchtower/).

## 7. (Optional) Expose Prometheus Metrics

In order to aid with monitoring, the TACo node can expose various metrics via [prometheus](https://prometheus.io/). Ursula can optionally provide a metrics endpoint as a data source for real-time monitoring.

The metrics endpoint is disabled by default but can be enabled by providing the following parameters to the `nucypher ursula run` command:

* `--prometheus` - a boolean flag to enable the prometheus endpoint
* `--metrics-port <PORT>` - the HTTP port to run the prometheus endpoint on. If not specified, the default is port `9101`.

{% hint style="warning" %}
The docker container will need to expose the specified port i.e. add `-p <PORT>:<PORT>` to the `docker run` command. For example, if the default port (`9101`) is used then add `-p 9101:9101`.
{% endhint %}

* `--metrics-interval <INTERVAL>` - the frequency of metrics collection in seconds. If not specified, the default is `90` seconds i.e. metrics are collected every 90 seconds.

{% hint style="warning" %}
In general, metrics collection will increase the number of RPC requests made to your provider endpoint; increasing the frequency of metrics collection will further increase this number.
{% endhint %}

The corresponding endpoint, `http://<node_ip>:<PORT>/metrics`, can be used as a prometheus data source.
