# SequentialCondition

Sequential conditions allow the execution of multiple conditions in order, where the outcome of one condition execution can be used by subsequent conditions. This concept is especially useful in workflows where intermediate results influence subsequent steps.

Each condition is evaluated in sequence, and if any condition in the sequence fails, the overall condition is considered as having failed.

## ConditionVariable

A sequential condition is comprised of a list of `ConditionVariable` objects.

The `ConditionVariable` object represents a condition whose result is tied to a specific variable name, which stores the result of the execution of the condition. This variable name can be referenced by subsequent conditions in the sequence.&#x20;

`ConditionVariables` have two mandatory attributes:

* `varName`: The name of the variable used to store the execution result of the condition.
* `condition`: Any [`Condition`](../../application-development/conditions/) to evaluate, and includes allows nesting of a `SequentialCondition` within itself.

For example:

<pre class="language-typescript"><code class="lang-typescript">import { conditions } from '@nucypher/taco';

const timeCondition = new conditions.base.time.TimeCondition({
<strong>  chain: 1,
</strong>  returnValueTest: {
    comparator: '>=',
    value: 1701428400,
  },
});

const conditionVariable = {
<strong>  varName: 'timeValue',
</strong>  condition: timeCondition,
}
</code></pre>

In this case, the condition variable will store the current block time obtained and used by the `timeCondition` in the `timeValue` variable. Variable names are considered [Custom Context Variables](../../application-development/conditions/conditioncontext-and-context-variables.md#context-variables) and can be referenced by using the string `:<varName>` i.e. `:timeValue` from this example, in subsequent conditions.

## Example

<pre class="language-typescript"><code class="lang-typescript">import { conditions } from '@nucypher/taco';

// first condition variable
const conditionA = new conditions.base.contract.ContractCondition({
  contractAddress: '0x1e988106FCe9647Bdf1E7877BF73cE8B0BAD5f97',
  method: 'methodForA',
  parameters: [':userAddress'],
  functionAbi: ...,
  chain: 1,
  returnValueTest: {
    comparator: '!=',
    value: "0x",
  },
});
<strong>const conditionVariableA = {
</strong>    varName: 'value_a',
    condition: conditionA,
}

// second condition variable
const conditionB = new conditions.base.contract.ContractCondition({
  contractAddress: '0x4838B106FCe9647Bdf1E7877BF73cE8Bc48AaD77',
  method: 'methodForB',
  parameters: [':value_a'],
  functionAbi: ...,
  chain: 1,
  returnValueTest: {
    comparator: '>',
    value: 0,
  },
});
const conditionVariableB = {
    varName: 'value_b',
    condition: conditionB,
}

// sequential condition
const sequentialCondition = new conditions.sequential.SequentialCondition({
  conditionVariables: [
     conditionVariableA,
     conditionVariableB,
  ],
});
</code></pre>

In this example, the sequential condition is evaluated as follows:

1. `conditionVariableA` is assessed by executing `conditionA`, which is a [`ContractCondition`](../contractcondition/).
   * `conditionA` will invoke `methodForA` on the relevant contract, passing the validated user address as an argument.
   * The result of this function is then checked to ensure it is not an empty byte.
   * If `conditionA` fails, the sequential condition is immediately marked as failed, and no further conditions are evaluated. If it succeeds, the result is stored as `:value_a`, and the evaluation proceeds to `conditionVariableB`.
2. `conditionVariableB`, also a [`ContractCondition`](../contractcondition/), invokes `methodForB` on the relevant contract, using the output from `conditionA` (`:value_a`) as the argument.
   * The result is verified to ensure it is not equal to 0.
   * If `conditionB` fails, the entire sequential condition is marked as failed. If it passes, the result is stored as `:value_b`.

Since no further condition variables remain and both `conditionA` and `conditionB` have passed, the sequential condition is considered satisfied.

{% hint style="info" %}
Individual conditions within the `SequentialCondition` use different chain IDs as long as the chain IDs are supported by the `domain` network being used.
{% endhint %}

## Code Changes

* Client-side: [https://github.com/nucypher/taco-web/pull/581](https://github.com/nucypher/taco-web/pull/581)
* Server-side: [https://github.com/nucypher/nucypher/pull/3500](https://github.com/nucypher/nucypher/pull/3500)
