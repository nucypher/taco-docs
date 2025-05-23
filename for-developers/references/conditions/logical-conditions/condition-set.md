# CompoundCondition

`CompoundConditon` allow conditions to be combined into more complex logical statements. The logical operators allowed are `or`, `and`, and `not`.

## `or`/`and` operators

`or` & `and` operators require >= 2 operands to be specified.

Let's take a look at this abbreviated condition example:

```typescript
import { conditions } from '@nucypher/taco';

const conditionA = ...
const conditionB = ...

const aAndB = new conditions.compound.CompoundCondition({
  operator: 'and',
  operands: [conditionA, conditionB],
});
```

Here, we define two conditions `conditionA` and `conditionB`, and combine them into a single condition using `and` operator. So now, our new condition will "trigger" only when both `conditionA` _and_ `conditionB` "trigger".

We can use two or more  conditions when using `and` and `or` operators:

```typescript
const allMustPass = new conditions.compound.CompoundCondition({
  operator: 'and',
  operands: [conditionA, conditionB, ..., conditionX],
});

const anyOneMustPass = new conditions.compound.CompoundCondition({
  operator: 'or',
  operands: [conditionA, conditionB, ..., conditionX],
});

```

Alternatively, we can use `CompoundCondition.or` and `CompoundCondition.and` short-hand methods

```typescript
const allMustMatch = new conditions.compound.CompoundCondition.and([
    conditionA, conditionB, ..., conditionX],
]);

const atLeastOneMustMatch = new conditions.compound.CompoundCondition.or([
    conditionA, conditionB, ..., conditionX],
]);
```

## `not` operator

The `not` operator only allows one operand.

For example:

```typescript
import { conditions } from '@nucypher/taco';

const conditionA = ...

const notA = new conditions.compound.CompoundCondition({
  operator: 'not',
  operands: [conditionA],
});
```

## Nested Combinations

`CompoundCondition` objects can be nested on itself i.e. you can create compound conditions of compound conditions.

For example:

```typescript
import { conditions } from '@nucypher/taco';

const conditionA = ...
const conditionB = ...
const conditionC = ...
const conditionD = ...
const conditionE = ...

const cOrD = new conditions.compound.CompoundCondition({
  operator: 'or',
  operands: [conditionC, conditionD],
});

const notE = new conditions.compound.CompoundCondition({
  operator: 'not',
  operands: [conditionE],
});

// conditionA AND conditionB AND cOrD AND notE
overallCondition = new conditions.compound.CompoundCondition({
  operator: 'and',
  operands: [conditionA, conditionB, cOrD, notE],
});
```

{% hint style="info" %}
Individual conditions within the `CompoundCondition`can use different chain IDs as long as the chain IDs are supported by the `domain` network being used.
{% endhint %}

## Learn more&#x20;

* [..](../../ "mention")
