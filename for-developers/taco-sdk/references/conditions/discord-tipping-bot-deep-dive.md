# Deep Dive: A Discord Tipping Bot Condition

This is one of the most intricate real-world conditions an integrator has built on TACo. Read it once and you have seen every major feature: nested compound logic, sequential variables, on-chain CREATE2 derivation, JSONPath extraction, snowflake arithmetic, ECDSA verification, and nested ABI calldata validation. Paste the whole page into an LLM's context when you want it to author conditions of comparable complexity.

The condition guards a TACo Action Control flow built by an integrator: a Discord user types `/tip 0.50 0xfriend` in a Discord server running a tipping bot, and a smart account derived deterministically from their Discord ID sends 0.50 USDC on Base — but **only** if every clause below is satisfied. The TACo network performs threshold signing on the resulting `UserOperation` exactly when the condition evaluates true.

> **Addresses anonymised.** Where you see `0xUSDC_ON_BASE`, `0xAA_FACTORY`, `0xACCOUNT_BYTECODE_HASH`, `<ED25519_PUBKEY>`, etc., substitute the real values from the [`discord-taco-web`](https://github.com/nucypher/discord-taco-web) repo. The structure is what matters.

## What the condition enforces

1. The Discord interaction payload was actually signed by a known relay's Ed25519 key (proves the request originated from Discord and passed through the trusted relay).
2. The Discord account is at least 6 months old (anti-spam).
3. The smart account submitting the `UserOperation` is the **deterministically derived** account for that exact Discord user — nobody can spend someone else's tip allowance.
4. The `UserOperation` calldata is exactly `execute(USDC, 0, transfer(recipient, amount))`, with the recipient and amount lifted from the Discord slash-command parameters and the amount converted to USDC base units.

If any of those fail, no signature.

## Top-level shape

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "compound",
    "operator": "and",
    "operands": [
      { "...": "1. ECDSA signature on Discord payload" },
      { "...": "2. SequentialCondition: account-age + AA derivation + sender check" },
      { "...": "3. SequentialCondition: parse tip params + ABI calldata validation" }
    ]
  }
}
```

A `CompoundCondition` with `and` over three operands. Two of those operands are themselves `SequentialCondition`s — that is how we get more than the 5-condition limit's worth of expressive power inside a compound.

## Clause 1: Verify the relay's signature

```json
{
  "conditionType": "ecdsa",
  "message": ":timestamp:discordPayload",
  "signature": ":signature",
  "verifyingKey": "<ED25519_PUBKEY>",
  "curve": "Ed25519"
}
```

When the bot relays a Discord interaction to TACo, it passes:

- `:timestamp` — Discord interaction timestamp
- `:discordPayload` — the full JSON of the slash-command interaction
- `:signature` — the Ed25519 signature the relay applied to `timestamp || discordPayload`
- `:verifyingKey` — the relay's well-known public key (a constant, baked into the condition)

If this fails, the network refuses to sign. Nobody can fake a Discord interaction without the relay's private key.

## Clause 2: Sequential — derive the AA and prove sender identity

This is the heart of the condition. Six steps, each binding a variable subsequent steps depend on.

```json
{
  "conditionType": "sequential",
  "conditionVariables": [
    { "varName": "senderId",                "condition": { "..." } },
    { "varName": "minAccountCreationTime",  "condition": { "..." } },
    { "varName": "validateAccountAge",      "condition": { "..." } },
    { "varName": "senderSalt",              "condition": { "..." } },
    { "varName": "senderAA",                "condition": { "..." } },
    { "varName": "validateSender",          "condition": { "..." } }
  ]
}
```

### Step 2.1 — Pull the Discord user ID

```json
{
  "varName": "senderId",
  "condition": {
    "conditionType": "json",
    "data": ":discordPayload",
    "query": "$.member.user.id",
    "returnValueTest": { "comparator": ">", "value": 0 }
  }
}
```

`:discordPayload` is supplied by the bot. JSONPath extracts `member.user.id` (a Discord snowflake — a 64-bit integer encoded as a string). The `> 0` test is a "not empty" sentinel; the real validation happens in step 2.3.

### Step 2.2 — Compute the cutoff timestamp (now − 6 months) in milliseconds

```json
{
  "varName": "minAccountCreationTime",
  "condition": {
    "chain": 8453,
    "method": "blocktime",
    "returnValueTest": { "comparator": ">", "value": 0 },
    "conditionType": "time"
  },
  "operations": [
    { "operation": "*=", "value": 1000 },
    { "operation": "-=", "value": 15768000000 }
  ]
}
```

`TimeCondition` returns Base's latest block timestamp in seconds. The two `operations` on the variable convert it to milliseconds (`*1000`) and subtract six months in ms (`15768000000`). The result is bound as `:minAccountCreationTime`.

> **Why operations on the variable, not on `returnValueTest`?** The `> 0` here is a no-op truthiness check — we just want the timestamp itself. The arithmetic transforms the **stored** value, not the comparison.

### Step 2.3 — Validate Discord snowflake → account age

```json
{
  "varName": "validateAccountAge",
  "condition": {
    "conditionType": "context-variable",
    "contextVariable": ":senderId",
    "returnValueTest": {
      "operations": [
        { "operation": "int" },
        { "operation": "/=", "value": 4194304 },
        { "operation": "+=", "value": 1420070400000 }
      ],
      "comparator": "<",
      "value": ":minAccountCreationTime"
    }
  }
}
```

This is the snowflake decode trick. Discord snowflakes are 64-bit integers where the upper 42 bits encode `(unix_ms - DISCORD_EPOCH)`. To get the account creation time:

1. `int` — coerce the snowflake string to integer
2. `/= 4194304` — right-shift by 22 bits (divide by `2^22`)
3. `+= 1420070400000` — add the Discord epoch (2015-01-01 in ms)

Then assert: creation time `<` `:minAccountCreationTime`. In English: "the account was created before the cutoff", i.e. it is at least 6 months old.

### Step 2.4 — Derive the salt for CREATE2

```json
{
  "varName": "senderSalt",
  "condition": {
    "conditionType": "context-variable",
    "contextVariable": ":senderId",
    "returnValueTest": { "comparator": ">", "value": 0 }
  },
  "operations": [
    { "operation": "str" },
    { "operation": "+=", "value": "|Discord|TipBot" },
    { "operation": "keccak" }
  ]
}
```

We take the Discord ID, stringify it, append `|Discord|TipBot`, and hash it with keccak. The result is the deterministic salt for the smart account derivation. Two different Discord users → two different salts → two different smart accounts. The same Discord user always derives the same salt.

### Step 2.5 — Compute the smart account address

```json
{
  "varName": "senderAA",
  "condition": {
    "chain": 8453,
    "method": "computeAddress",
    "parameters": [
      "0xACCOUNT_BYTECODE_HASH",
      ":senderSalt"
    ],
    "contractAddress": "0xAA_FACTORY",
    "functionAbi": {
      "name": "computeAddress",
      "type": "function",
      "stateMutability": "view",
      "inputs": [
        { "name": "_bytecodeHash", "type": "bytes32", "internalType": "bytes32" },
        { "name": "_salt",         "type": "bytes32", "internalType": "bytes32" }
      ],
      "outputs": [
        { "name": "", "type": "address", "internalType": "address" }
      ]
    },
    "returnValueTest": {
      "comparator": "!=",
      "value": "0x0000000000000000000000000000000000000000"
    },
    "conditionType": "contract"
  }
}
```

A real on-chain `view` call into the AA factory. Given the bytecode hash (constant) and the salt (computed in 2.4), the factory returns the CREATE2 address — the smart account that *would* exist (or already exists) for this Discord user. We bind it as `:senderAA`.

> **Alternative:** the schema also exposes a `create2` operation that computes the address **locally** without an RPC round-trip. The bot uses the on-chain version because the factory's address-derivation logic may evolve; a local computation would have to be kept in sync.

### Step 2.6 — Assert the UserOperation `sender` matches

```json
{
  "varName": "validateSender",
  "condition": {
    "signingObjectContextVar": ":signingConditionObject",
    "attributeName": "sender",
    "conditionType": "signing-attribute",
    "returnValueTest": {
      "comparator": "==",
      "value": ":senderAA"
    }
  }
}
```

`:signingConditionObject` is the `UserOperation` the network is being asked to sign. Read its `sender` field and require it to equal the address we derived in step 2.5.

This is the linchpin. Without this clause, anyone could submit a `UserOperation` from *any* smart account and ride on the rest of the validation. With it, the only smart account that can ever pass is the one deterministically tied to that Discord user.

## Clause 3: Sequential — parse tip params and validate calldata

```json
{
  "conditionType": "sequential",
  "conditionVariables": [
    { "varName": "amountUSDC",       "condition": { "..." } },
    { "varName": "recipientDirect",  "condition": { "..." } },
    { "varName": "validateCalldata", "condition": { "..." } }
  ]
}
```

### Step 3.1 — Extract and normalise the tip amount

```json
{
  "varName": "amountUSDC",
  "condition": {
    "conditionType": "json",
    "data": ":discordPayload",
    "query": "$.data.options[0].options[?(@.name == \"amount\")].value",
    "returnValueTest": {
      "comparator": ">=",
      "value": 0.25,
      "operations": [
        { "operation": "float" }
      ]
    }
  },
  "operations": [
    { "operation": "toTokenBaseUnits", "value": 6 }
  ]
}
```

A JSONPath filter (`?(@.name == "amount")`) finds the slash-command argument named `amount`. The return-value test coerces it to a float and asserts a 0.25 USDC minimum. The variable-level `operations` then multiply the float by `10^6` to get USDC base units, and bind that as `:amountUSDC`.

### Step 3.2 — Extract the recipient address

```json
{
  "varName": "recipientDirect",
  "condition": {
    "conditionType": "json",
    "data": ":discordPayload",
    "query": "$.data.options[0].options[?(@.name == \"address\")].value",
    "returnValueTest": { "comparator": ">", "value": 0 }
  }
}
```

Same trick — pull the `address` argument out of the slash-command payload, bind as `:recipientDirect`.

### Step 3.3 — Validate the UserOperation calldata exactly

```json
{
  "varName": "validateCalldata",
  "condition": {
    "signingObjectContextVar": ":signingConditionObject",
    "attributeName": "call_data",
    "conditionType": "signing-abi-attribute",
    "abiValidation": {
      "allowedAbiCalls": {
        "execute((address,uint256,bytes))": [
          {
            "parameterIndex": 0,
            "subIndices": [0],
            "returnValueTest": {
              "comparator": "==",
              "value": "0xUSDC_ON_BASE"
            }
          },
          {
            "parameterIndex": 0,
            "subIndices": [1],
            "returnValueTest": { "comparator": "==", "value": 0 }
          },
          {
            "parameterIndex": 0,
            "subIndices": [2],
            "nestedAbiValidation": {
              "allowedAbiCalls": {
                "transfer(address,uint256)": [
                  {
                    "parameterIndex": 0,
                    "returnValueTest": {
                      "comparator": "==",
                      "value": ":recipientDirect"
                    }
                  },
                  {
                    "parameterIndex": 1,
                    "returnValueTest": {
                      "comparator": "==",
                      "value": ":amountUSDC"
                    }
                  }
                ]
              }
            }
          }
        ]
      }
    }
  }
}
```

This is the most intricate ABI validation TACo supports today. Walking it:

- The `UserOperation`'s `call_data` must decode as `execute((address,uint256,bytes))` — the standard ERC-4337 single-call entry point. It takes one tuple parameter.
- `parameterIndex: 0, subIndices: [0]` — the first field of the tuple (the `address` target) must equal the USDC contract on Base. **No other token can be transferred.**
- `parameterIndex: 0, subIndices: [1]` — the tuple's `value` field must be 0 (no native ETH attached).
- `parameterIndex: 0, subIndices: [2]` — the tuple's `bytes` field is itself ABI-encoded calldata. We decode it with `nestedAbiValidation` and assert it is exactly `transfer(address,uint256)` with:
  - parameter 0 (`address`) equal to `:recipientDirect` — the recipient extracted from the Discord payload in step 3.2
  - parameter 1 (`uint256`) equal to `:amountUSDC` — the normalised amount from step 3.1

The chain of inference: Discord said tip 0.50 USDC to `0xfriend` → bot built a `UserOperation` that calls `execute(USDC, 0, transfer(0xfriend, 500_000))` → TACo signs it only if the calldata literally matches what the Discord message asked for. **Front-running, calldata mutation, or token substitution all fail.**

## What this example teaches

- **Compound + sequential is how you build deep logic.** The 5-condition compound limit is not a limit on expressivity if you embed sequentials inside the operands.
- **Sequential variables are the workhorse.** Use them whenever a check depends on a value computed elsewhere in the condition.
- **`operations` lets you do real arithmetic on-chain-ish** — bit shifts via division, hashing, type coercion, unit conversions. Many things you would otherwise need a smart contract for can be done in the condition itself.
- **`signing-abi-attribute` with `nestedAbiValidation`** is how you enforce policy on ERC-4337 calldata structure. This is the killer feature for Action Control.
- **Anonymise constants only when necessary.** In a production deployment the bytecode hash, factory address, and relay verifying key are all hardcoded and public. They are anonymised on this page for documentation hygiene; an integrator's own conditions normally do not need to be.

## Want to run this?

The full, un-anonymised condition lives at [`discord-taco-web/conditions-mainnet.json`](https://github.com/nucypher/discord-taco-web/blob/main/conditions-mainnet.json). Validate it locally with the [`validate-conditions.ts`](validating-conditions.md) script before shipping any modifications.
