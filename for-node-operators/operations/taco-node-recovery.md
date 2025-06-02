# TACo Node Recovery

## Node Operator Security and Recovery Guidelines

As a node operator, it is critical to maintain the security of your keystores (private keys), passwords, and mnemonic phrases throughout the lifecycle of your node's operation. Ensuring the safekeeping of these elements is essential for continued access and control of your node. In the event of a loss, several built-in recovery tools are available to assist you in restoring normal operations.

Below are the three possible high-level recovery scenarios:

1. **Recovery using a backup of keystore and password**
2. **Recovery using mnemonic**
3. **Complete loss of keystore and mnemonic**

This documentation outlines the procedures to manage scenarios 1 and 2. However, please be advised that in the case of a complete loss of both the keystore and mnemonic, there are currently no recovery options available and you will need to shut down your node until a re-onboarding mechanism is included in a future software upgrade (this will result in reward withholding and/or stake slashing).

If you find yourself in this situation, please reach out for assistance by opening a support ticket in the Threshold Discord server under the `#support-ticket` channel.

### Recovery

**Recover a TACo node using a mnemonic and existing config**

This command can be used to restore private keys on an existing node.

{% hint style="info" %}
If using Docker commands, start by pulling the latest recovery image:

```bash
docker pull nucypher/nucypher:recovery
```
{% endhint %}

{% tabs %}
{% tab title="CLI" %}
```bash
nucypher ursula recover
```
{% endtab %}

{% tab title="Docker" %}
```bash
docker run -it -v ~/.local/share/nucypher:/root/.local/share/nucypher:rw -v ~/.ethereum/:/root/.ethereum:ro nucypher/nucypher:recovery nucypher ursula recover
```
{% endtab %}
{% endtabs %}

#### **Recover or relocate a TACo node by creating a new configuration with mnemonic**

This command can be used to completely relocate a node to a new host from scratch while preserving the original private keys.

{% tabs %}
{% tab title="CLI" %}
```bash
nucypher ursula init ... --with-mnemonic
```
{% endtab %}

{% tab title="Docker" %}
```bash
docker run -it -v ~/.local/share/nucypher:/root/.local/share/nucypher:rw -v ~/.ethereum/:/root/.ethereum:ro nucypher/nucypher:recovery nucypher ursula init ... --with-mnemonic
```
{% endtab %}
{% endtabs %}

### **View public keys for a given mnemonic**

This command is useful if you have a mnemonic but are unsure which public keys it produces.

{% tabs %}
{% tab title="CLI" %}
```bash
nucypher ursula public-keys --from-mnemonic
```
{% endtab %}

{% tab title="Docker" %}
```bash
docker run -it nucypher/nucypher:recovery nucypher ursula public-keys --from-mnemonic
```
{% endtab %}
{% endtabs %}

### Auditing <a href="#auditing" id="auditing"></a>

Below is documentation for node auditing commands that can be used to ensure correctness of passwords and mnemonics.Comment

#### **Audit password and mnemonic**

{% tabs %}
{% tab title="CLI" %}
```bash
nucypher ursula audit

nucypher ursula audit --config-file <config path>

nucypher ursula audit --keystore-filepath <keystore path>
```
{% endtab %}

{% tab title="Docker" %}
```bash
docker run -it -v ~/.local/share/nucypher:/root/.local/share/nucypher:rw -v ~/.ethereum/:/root/.ethereum:ro nucypher/nucypher:recovery nucypher ursula audit
```
{% endtab %}
{% endtabs %}

#### **View mnemonic**

This command can be used to view the mnemonic for existing private keys.  This assumes you have the keystore file and it's associated password.

{% tabs %}
{% tab title="CLI" %}
```bash
nucypher ursula audit ... --view-mnemonic
```
{% endtab %}

{% tab title="Docker" %}
```bash
docker run -it -v ~/.local/share/nucypher:/root/.local/share/nucypher:rw -v ~/.ethereum/:/root/.ethereum:ro nucypher/nucypher:recovery nucypher ursula audit --view-mnemonic
```
{% endtab %}
{% endtabs %}
