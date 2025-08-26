# Conditions

This section focuses on `Condition` types, composition and usage.

## Base Conditions

Base conditions define specific criteria, and each includes a `returnValueTest` to compare the actual execution result with the expected value. These include:

* [`TimeCondition`](timecondition.md) – time-based conditions using block height and other blockchain-based timestamps. \
  &#xNAN;_&#x45;xample:_ only allow access after a certain timestamp.
* [`RpcCondition`](rpccondition.md) –  based on RPC calls as defined in Ethereum's Official [API](https://ethereum.org/en/developers/docs/apis/json-rpc/#json-rpc-methods). \
  &#xNAN;_&#x45;xample:_ allow access if the requestor address holds a minimum ETH balance.
* [`ContractCondition`](contractcondition/) – uses on-chain state, allowing arbitrary contract function calls. \
  &#xNAN;_&#x45;xample:_ allow access if this requestor holds a special-purpose NFT.&#x20;
* [JSON Endpoint Conditions](json-endpoint-conditions/) - uses state from a JSON HTTPS endpoint.\
  &#xNAN;_&#x45;xample:_ allow discount on event tickets/merchandise if there is "bad" weather according to a specific weather API.
* &#x20;[`JWTCondition`](jwtcondition.md) - validates [JSON Web Tokens (JWTs)](https://datatracker.ietf.org/doc/html/rfc7519) against a specified public key.
* [Signing Object Conditions](signing-object-conditions.md) - validates objects submitted for threshold signing meet specified criteria by inspecting the values of designated fields (eg. `UserOperation` for ERC-4337)

Each base condition defines a [`returnValueTest`](./#return-value-test) used to compare the obtained execution value with the expected value for the condition.

### returnValueTest

A `returnValueTest` is a mechanism used by a condition to evaluate whether a specific execution result meets a specified criterion. It allows dynamic comparisons between the actual returned value and the expected value.&#x20;

It consists of three key components:&#x20;

* `comparator`: defines the comparison operation to apply between the actual value obtained and the expected value. The available operators include:
  * `==`: equal to
  * `!=`: not equal to
  * `>`: greater than
  * `<`: less than
  * `>=`: greater than or equal to
  * `<=`: less than or equal to
  * `in`: value in array of values
  * `!in`: value not in array of values
* `value`: the expected value to compare against the actual returned value.
* `index` _(optional)_: indicates the position of the value to use for comparison within a list or array when multiple values are returned. If the response includes several values, this index determines which entry to evaluate. If the index is not specified, the entire response is used. For instance, if the array `["apple", "banana", "grape"]` is returned during execution, an index of `1` would select `"banana"` as the value for comparison.

### Context Variables

[Context variables](../authentication/conditioncontext-and-context-variables.md) provide the ability for placeholder values to be defined within conditions at creation time, and be dynamically populated at verification time e.g. current user wallet address.

## Logical Conditions

Logical conditions use control structures to determine overall condition outcomes based on the results of underlying conditions. These include:

* [`CompoundCondition`](logical-conditions/condition-set.md) - allows conditions to be combined using logical operators such as `or`, `and` & `not` .
* [`SequentialCondition`](logical-conditions/sequentialcondition.md) - chains conditions to be executed in a specific order, where the outcome of one condition can be used by subsequent conditions.
* [`IfThenElseCondition`](logical-conditions/ifthenelsecondition.md) - implements branching logic for conditions where the flow follows an if-then-else structure i.e. **IF** `CONDITION_A` **THEN** `CONDITION_B` **ELSE** `CONDITION_C`

{% hint style="info" %}
Since condition evaluations may require making remote calls (e.g. RPC calls, etc.), the number of conditions allowed within a `Logical Condition` is limited.&#x20;

A single `Logical Condition`can contain a maximum of five conditions. Additionally, the nesting of  a `Logical Condition` within a `Logical Condition` is allowed, but the maximum nesting depth is restricted to two levels. Therefore, a `Logical Condition` can contain a sub-`Logical Condition` but that sub-`Logical Condition` cannot subsequently contain a `Logical Condition`.
{% endhint %}
