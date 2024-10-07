# Access Control

This section focuses on `Condition` types, composition and usage.

## Base Conditions

* [`TimeCondition`](timecondition.md) – time-based conditions using block height and other blockchain-based timestamps. E.g. allow access for one week after the first request is made.&#x20;
* [`RpcCondition`](../../conditions/rpccondition.md) –  based on RPC calls as defined in Ethereum's Official API [documentation](https://ethereum.org/en/developers/docs/apis/json-rpc/#json-rpc-methods). E.g. allow access if the address held ETH before 2020.&#x20;
* [`ContractCondition`](../../conditions/contractcondition/) – based on on-chain state, and can be any arbitrary contract function call. E.g. allow access if this requestor holds a special-purpose NFT.&#x20;

Each base condition defines a `returnValueTest` used to compare the obtained execution value with the expected value for the condition.

## Multi-Conditions

* [`CompoundCondition`](condition-set.md) - access conditions can be logically combined using the `or`, `and` & `not` operators.
* [`SequentialCondition`](../../conditions/future-enhancements/sequentialcondition.md) - access conditions can be chained and executed in order where the outcome of one condition execution can be used by subsequent conditions.
* [`IfThenElseCondition`](../../conditions/future-enhancements/ifthenelsecondition.md) - uses branching logic for access conditions i.e. **IF** `CONDITION A` **THEN** `CONDITION B` **ELSE** `CONDITION_C`

{% hint style="info" %}
Since condition evaluations may require making remote calls (e.g. RPC calls, etc.), the number of conditions allowed within a `Multi-Condition` is limited.&#x20;

A single `Multi-Condition` can contain a maximum of five conditions. Additionally, nesting of  a `Multi-Condition` within a `Multi-Condition` is allowed, but the maximum nesting depth is restricted to two levels. Therefore, a `Multi-Condition` can contain a sub-`Multi-Condition` but the sub-`Multi-Condition` cannot subsequently contain a `Multi-Condition`.
{% endhint %}

## Context Variables

[Context variables](conditioncontext-and-context-variables.md) provide the ability for placeholder values to be defined within conditions at encryption time, and be dynamically populated at decryption time e.g. current user wallet address.

{% hint style="info" %}
Multiple helper objects are provided to streamline the creation of conditions. An expressive API allows for granular control of conditions and examples of methods wherever possible.
{% endhint %}
