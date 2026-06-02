# Duties, Compensation & Penalties



{% hint style="warning" %}
The TACo network is currently dormant ahead of a relaunch by WEDF scheduled for Q3 2026. Although  independent operators may choose to continue running TACo nodes at their discretion, there is no coordinated group of providers running TACo software at this time, **nor are there any test DKG rituals.**

A stable version of the service will be relaunched in Q3 2026 – centered around a _Privacy Coalition._ If you represent an organization in the domains of privacy advocacy, anti-surveillance, human rights, encryption/whistleblower technology, or pertinent academic research, and would like to be involved, please get in [touch](https://discord.gg/Rh2728Hk).&#x20;

Until then, this page serves as a reference for prospective members of this new node coalition.&#x20;
{% endhint %}

### Node **Operator Duties**

{% hint style="danger" %}
Failure to adhere to node operator duties may result in on-chain punishments, including the withholding of rewards and slashing of stakes. See Violations & Penalties section below.
{% endhint %}

Operating a TACo node requires active engagement. To provide high-quality service and maintain the reliability of the Threshold Network, TACo node operators must:

1. **Safeguard TACo private Keys and secret mnemonic**: This includes creating an off-site record of the mnemonic assigned to your node in the initialization step and securing a copy of the keystore directory and passwords. Loss of the keystore (private keys) and password, or mnemonic will result in reward withholding and/or penalties.
2. **Maintain continuous server accessibility**: Ensure the server or machine running TACo software is online and accessible at all times, allowing for immediate verification and response to incoming decryption requests. This prevents delays in reaching a threshold and providing decryption material to qualifying data recipients. Continuous downtime or network unavailability will result in reward withholding and/or penalties.
3. **Update to the latest TACo version**: Ensure your node is running the latest version of TACo. New releases will be announced on the Threshold Discord #announcements channel; enable notifications to stay updated. Running an outdated version of TACo will result in reward withholding and/or penalties.

### Violations & Penalties

{% hint style="warning" %}
**As of September 2024, violations are being detected and logged on-chain via the Infraction Collector contract.**

Today, avoiding committing attributable violations increases the likelihood that a node is selected by TACo's adopting developers to participate in DKG Initialization Rituals, and hence form part of TACo cohorts managing access to real-world data.

Reward withholding and stake slashing, as a consequence of committing attributable violations, are not yet in effect, but will be rolled out in the near future.
{% endhint %}

Stakers operating TACo nodes must ensure that their nodes are up-to-date, available/reachable and correctly configured at all times. Failure to do may result in protocol divergent behavior, attributable failures, and the levying of economic penalties, including reward withholding and slashing of collateral.

<figure><img src="../.gitbook/assets/file.excalidraw (2).svg" alt=""><figcaption></figcaption></figure>

Nodes are 'tested' during Distributed Key Generation (DKG) rituals, wherein the following violations are detectable, attributable and punishable on-chain:

* Node is unreachable, non-responsive or offline.
* Node fails to submit a valid transcript or the transcript is missing.
* Node's keystore is mismatched with the ritual-coordinating smart contract. This implies the node operator has reset their node without following the correct recovery path, which is a protocol violation. If you have reset your node, please file a [support ticket](https://discord.com/channels/866378471868727316/1025113672185552938) – noting that resetting a TACo node (for any reason) is categorized as node mismanagement and may incur on-chain economic penalties in the future.

The following DKG ritual failure modes are considered outside of the node operator's control and currently do not incur any penalties:

* DKG ritual fails to generate a valid aggregation of transcripts.
* DKG ritual fails or times out due to blockchain or web3 provider issues.
* DKG ritual failure due to a bug in a node client release.

DKG rituals can occur at any time. There are two types of rituals:

(1) **Initialization rituals.** These are DKG rituals initiated by TACo's adopting developers – the first step for a developer to integrate TACo access control in production and control their own cohort of nodes.

(2) **Test & Heartbeat rituals.** These are manually initiated and automated dummy rituals, respectively. These generate statistics on the health of the network and flush out errant nodes ahead of Initialization rituals.

DKG rituals are initiated without any warning – it is almost impossible to predict when a DKG ritual will occur, when your TACo node will be called into action, and when a violation could be detected and attributed. It is therefore safest to have your TACo node up-to-date, available/reachable, and correctly configured at all times.

{% hint style="info" %}
All nodes must successfully complete several test/heartbeat rituals before being selected for an Initialization ritual with an adopting developer.
{% endhint %}

The following parameters serve as a guide for economic penalties that will deployed in forthcoming versions of TACo. The first three violations committed by a node will be penalized via the withholding of monthly T rewards, with escalating severity for repeat offenses. If more than three violations occur, then the node operator will have a part of their stake slashed.\
\
**1st** violation: **30%** rewards withholding for 3 months.\
**2nd** violation: **60%** rewards withholding for 3 months.\
**3rd** violation: **90%** rewards withholding for 3 months.\
**4th** violation: (% TBD) slashing of stake, one-off.\
\
Note that the violation count is reset if a node manages to avoid committing any further violations during a given 3 month 'penalty period'. However, every violation restarts the penalty period. This means that, firstly, the duration over which reward withholding is levied is always three months from the date of that violation. Secondly, the violation count, and corresponding severity of punishment, will continue to ratchet up with each offense, until a node manages to avoid committing a violation for three months.

{% hint style="info" %}
**Note for developers using TACo**\
Critical operations required for encryption & decryption follow a threshold design, hence isolated errant behavior will not disrupt end-user data sharing. However, DKG rituals, which are prerequisite for usage of TACO, are less fault tolerant and can fail if a single selected node fails to follow the protocol, inadvertently or otherwise. Hence, TACo's disincentive protocol is centered around DKG rituals, which (1) generate on-chain evidence for faults and (2) flush out negligent or bad actors before they can affect end-users.
{% endhint %}

### **Deauthorization Delay**

Adopters of TACo require reassurance that the cohorts of nodes managing their users' data will remain intact for extended durations. Although cohort members can be securely replaced, while maintaining a persistent public key, it is preferable for economic reasons to minimize the number of node replacement rituals – particularly in the genesis era of the service. Hence, TACo service provision ideally involves a commitment to provide service for years, rather than months.

{% hint style="warning" %}
The _deauthorization del&#x61;_&#x79; is the time one must wait between initiating a withdrawal from TACo service provision and being able to complete that withdrawal. From genesis, the deauthorization delay is set to **6 months** (183 days).
{% endhint %}

Note that this delay is universal and independent of any other token lock-up or bonus mechanism.
