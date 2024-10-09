# IfThenElseCondition

The `IfThenElseCondition` allows for conditional branching, where different conditions are executed based on the outcome of an initial "if" condition. It operates with a simple "if-then-else" structure and provides flexibility in determining the behaviour when a condition is true or false.

A condition that allows for if-then-else branching based on underlying conditions\
i.e. **IF** `CONDITION_A` **THEN** `CONDITION_B` **ELSE** `CONDITION_C`.

It is composed of:

* `ifCondition`: A required field that specifies the condition to be evaluated first. If the condition evaluates to `true`, the `thenCondition` is executed; if it evaluates to `false`, the `elseCondition` is executed.
* `thenCondition`: A required field that specifies the action or condition to execute if the `ifCondition` evaluates to `true`.
* `elseCondition`: A required field that specifies the action or condition to execute if the `ifCondition` evaluates to `false`. This can be a `CONDITION`or a boolean value (`true` or `false`), specifying the result when the `ifCondition` is `false`. This gives users control over what the overall condition returns when the `ifCondition` is false.

When combined with other conditions (e.g., `SequentialCondition`, `CompoundCondition`), the `IfThenElseCondition` allows for complex, branching workflows.

## Examples

### IF `conditionA` then `conditionB` else `conditionC`

```typescript
import { conditions } from '@nucypher/taco';

const conditionA = ...
const conditionB = ...
const conditionC = ...

const condition = new conditions.ifThenElse.IfThenElseCondition({
  ifCondition: conditionA,
  thenCondition: conditionB,
  elseCondition: conditionC,
});
```

### IF `conditionA` then `conditionB` else True

```typescript
import { conditions } from '@nucypher/taco';

const conditionA = ...
const conditionB = ...

const condition = new conditions.ifThenElse.IfThenElseCondition({
  ifCondition: conditionA,
  thenCondition: conditionB,
  elseCondition: true,
});
```

### Nested `IfThenElseCondition`

```typescript
import { conditions } from '@nucypher/taco';

const conditionA = ...
const conditionB = ...
const conditionC = ...
const conditionD = ...

const nestedIfThenElseCondition = new conditions.ifThenElse.IfThenElseCondition({
  ifCondition: conditionA,
  thenCondition: conditionB,
  elseCondition: false,
});

const condition = new conditions.ifThenElse.IfThenElseCondition({
  ifCondition: conditionC,
  thenCondition: conditionD,
  elseCondition: nestedIfThenElseCondition,
});
```

## Development References

* Client-side: [https://github.com/nucypher/taco-web/pull/593](https://github.com/nucypher/taco-web/pull/593)
* Server-side: [https://github.com/nucypher/nucypher/pull/3558](https://github.com/nucypher/nucypher/pull/3558)
