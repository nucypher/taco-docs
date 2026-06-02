# Trust Model

This page explains the principles behind the overall trust model, considers the more subjective or domain-specific aspects of the model with respect to risk and security, and touches upon potential future features that would reduce the trust burden and provide more optionality for developers and end-users.

### Cohort-related trust assumptions

The foundation of the TACo trust model – and much of threshold cryptography in general – is the concept of a _Cohort;_ a group of nodes employed to:

1. Collectively generate and manage part of a public key that can be used to encrypt one or more data payloads.
2. Collectively manage a decryption fragment associated with a single data payload and provide the fragment to requesters who fulfill pre-specified conditions.
3. Collective manage s signing key used for generating signatures for signing requests

All Cohorts are parametrized on formation, including the Cohort _size_ (`n`) and Cohort _threshold_ (`m`). These parameters are the inputs for the following core trust assumptions:

#### (1) Orderly Threshold

The first is the _orderly threshold_ assumption, wherein the protocol relies on a minimum number – the threshold – of node operators within each Cohort to follow the protocol correctly. For example, a 16-of-32 cohort would require at least **16** nodes to be online, responsive, and run an up-to-date version of TACo software. If any fewer than 16 are online, data requesters will be unable to retrieve decryption fragments.

#### (2) Honest Threshold

The second is the _honest threshold_ assumption, the protocol's most fundamental form of collusion-resistance – that is, protection against deliberate, unlawful attempts to access private data. In this case, the protocol relies on a minimum number of nodes to be ‘honest’ – i.e. not susceptible to bribery, coercion,, or other attempts to maliciously collude. This minimum is calculated as the threshold node count (`m`) subtracted from the total cohort size, plus one (`n - m + 1`). Using the same example as before, a 16-of-32 Cohort would require a minimum of 32 - 16 + 1 = **17** honest nodes. In other words, if at least 17 individual operators refuse to collude, there is nothing the remaining 15 nodes can do, regardless of their war chest or aggregate deposit power.

Note that the _orderly threshold_ and _honest threshold_ assumptions are conceptually similar to the more common _honest majority_ assumption. However, they are more flexible than simply requiring those who control 67% of the deposited collateral to be honest. Unlike most BFT or pBFT-based protocols, the _honest threshold_ can be partially decoupled from the nodes wealth, depending on the cohort sampling parameters specified by the developer or end-user (see next section).

### Nodes sampling-related trust assumptions

The _orderly threshold_ and _honest threshold_ trust assumptions above treat each Cohort as an isolated group, where the chosen parameters (`m-of-n`) determine the group’s redundancy, latency, and collusion resistance. However, the reality is that each Cohort is selected from a larger sample of Threshold nodes, which is larger than the typical/optimal size of each cohort.\
\
Therefore, the mechanisms through which nodes are selected to form Cohorts carry their own trust assumptions. More specifically, the _sampling parameters_ impact the security and collusion-resistance of a given data-sharing flow. Sampling parametrization can be divided into; (1) those relating to frequency and prompting of (re-)sampling, and (2) compositional requirements to form a Cohort, besides the top-level `m` & `n` parameters.

