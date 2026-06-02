# Condition Cookbook

A working JSON example for every condition type the TACo SDK supports, plus several real-world combinations. Copy, paste, modify.

All addresses below are **placeholders**. Substitute your own. All chain IDs are real (Ethereum=`1`, Polygon=`137`, Base=`8453`, Arbitrum=`42161`).

> Pair this page with the [schema reference](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md) and the [validator script](validating-conditions.md). Together they are everything an LLM needs to author conditions for you.
>
> **Editor tip:** add `"$schema": "https://raw.githubusercontent.com/nucypher/taco-web/signing-epic/packages/taco/schema-docs/condition-schema.json"` to the top of any `conditions.json` and your editor will validate every example below in-place. See [Building Conditions with an LLM → JSON Schema integration](building-with-llms.md#json-schema-integration).

## Table of contents

**Single conditions**

1. [TimeCondition — gate after a timestamp](#1-timecondition--after-a-given-time)
2. [RpcCondition — minimum native ETH balance](#2-rpccondition--minimum-eth-balance)
3. [ContractCondition — ERC-20 minimum balance (standard)](#3-contractcondition--erc-20-balance-standard-shorthand)
4. [ContractCondition — ERC-721 ownership (standard)](#4-contractcondition--erc-721-ownership)
5. [ContractCondition — custom view function with `functionAbi`](#5-contractcondition--custom-view-function)
6. [ContractCondition — allowlist mapping lookup](#6-contractcondition--allowlist-mapping-lookup)
7. [JsonApiCondition — public weather API](#7-jsonapicondition--public-weather-api)
8. [JsonApiCondition — authenticated price feed](#8-jsonapicondition--authenticated-api-with-bearer-token)
9. [JsonRpcCondition — arbitrary JSON-RPC call](#9-jsonrpccondition--arbitrary-json-rpc)
10. [JsonCondition — querying a passed-in payload](#10-jsoncondition--querying-a-context-payload)
11. [JwtCondition — verify a signed JWT](#11-jwtcondition--verify-a-jwt)
12. [EcdsaCondition — verify an ECDSA signature](#12-ecdsacondition--verify-an-ecdsa-signature)
13. [SigningObjectAttributeCondition — UserOperation field check](#13-signingobjectattributecondition--useroperation-field-check) (TACo Action Control only)
14. [SigningObjectAbiAttributeCondition — UserOperation calldata check](#14-signingobjectabiattributecondition--validate-calldata) (TACo Action Control only)
15. [ContextVariableCondition — assert a custom variable](#15-contextvariablecondition--assert-a-custom-variable)

**Logical / composed conditions**

16. [CompoundCondition — AND, OR, NOT](#16-compoundcondition--and-or-not)
17. [SequentialCondition — derive a value, then check it](#17-sequentialcondition--derive-then-check)
18. [IfThenElseCondition — branch on a condition](#18-ifthenelsecondition--branching)
19. [Pattern: NFT-or-allowlist gate](#19-pattern-nft-or-allowlist-gate)
20. [Pattern: time-windowed paid access](#20-pattern-time-windowed-paid-access)
21. [Pattern: storm-condition open access](#21-pattern-storm-condition-open-access)
22. [Pattern: balance fetched, normalised, then asserted](#22-pattern-balance-fetched-normalised-then-asserted)

For a worked end-to-end example that combines many features at once, see the [Discord tipping bot deep-dive](discord-tipping-bot-deep-dive.md).

---

## 1. `TimeCondition` — after a given time

Decryption is allowed only after the latest block on Polygon has a timestamp greater than `1735689600` (2025-01-01 UTC).

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "time",
    "chain": 137,
    "method": "blocktime",
    "returnValueTest": {
      "comparator": ">",
      "value": 1735689600
    }
  }
}
```

## 2. `RpcCondition` — minimum ETH balance

Allow if the requester holds at least 0.1 ETH on Ethereum mainnet.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "rpc",
    "chain": 1,
    "method": "eth_getBalance",
    "parameters": [":userAddress", "latest"],
    "returnValueTest": {
      "comparator": ">=",
      "value": 100000000000000000
    }
  }
}
```

`:userAddress` is automatically replaced with the requester's wallet address. The value `100000000000000000` is 0.1 ETH in wei.

## 3. `ContractCondition` — ERC-20 balance (standard shorthand)

Require at least 1000 USDC (6 decimals) on Base.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "contract",
    "chain": 8453,
    "contractAddress": "0xUSDC_ADDRESS_ON_BASE",
    "standardContractType": "ERC20",
    "method": "balanceOf",
    "parameters": [":userAddress"],
    "returnValueTest": {
      "comparator": ">=",
      "value": 1000000000
    }
  }
}
```

When `standardContractType` is set to `"ERC20"` or `"ERC721"`, the SDK uses the canonical ABI — you do not need to supply `functionAbi`.

## 4. `ContractCondition` — ERC-721 ownership

Hold at least one NFT from the specified collection on Polygon.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "contract",
    "chain": 137,
    "contractAddress": "0xNFT_COLLECTION_ADDRESS",
    "standardContractType": "ERC721",
    "method": "balanceOf",
    "parameters": [":userAddress"],
    "returnValueTest": {
      "comparator": ">",
      "value": 0
    }
  }
}
```

## 5. `ContractCondition` — custom view function

Call any `view` or `pure` function on any contract by supplying a `functionAbi`. Here we check that the user is a member of a DAO.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "contract",
    "chain": 1,
    "contractAddress": "0xDAO_MEMBERSHIP_CONTRACT",
    "method": "isMember",
    "parameters": [":userAddress"],
    "functionAbi": {
      "name": "isMember",
      "type": "function",
      "stateMutability": "view",
      "inputs": [
        { "name": "account", "type": "address", "internalType": "address" }
      ],
      "outputs": [
        { "name": "", "type": "bool", "internalType": "bool" }
      ]
    },
    "returnValueTest": {
      "comparator": "==",
      "value": true
    }
  }
}
```

## 6. `ContractCondition` — allowlist mapping lookup

A contract exposes `mapping(address => uint8) public tier;`. Allow only users in tier 2 or higher.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "contract",
    "chain": 8453,
    "contractAddress": "0xTIER_REGISTRY",
    "method": "tier",
    "parameters": [":userAddress"],
    "functionAbi": {
      "name": "tier",
      "type": "function",
      "stateMutability": "view",
      "inputs": [
        { "name": "", "type": "address", "internalType": "address" }
      ],
      "outputs": [
        { "name": "", "type": "uint8", "internalType": "uint8" }
      ]
    },
    "returnValueTest": {
      "comparator": ">=",
      "value": 2
    }
  }
}
```

## 7. `JsonApiCondition` — public weather API

Allow if today's temperature in London is below 10°C.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "json-api",
    "endpoint": "https://api.open-meteo.com/v1/forecast",
    "parameters": {
      "latitude": 51.5072,
      "longitude": -0.1276,
      "current": "temperature_2m"
    },
    "query": "$.current.temperature_2m",
    "returnValueTest": {
      "comparator": "<",
      "value": 10
    }
  }
}
```

## 8. `JsonApiCondition` — authenticated API with Bearer token

The decrypter supplies an API token at decryption time via `:apiToken`.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "json-api",
    "endpoint": "https://api.example.com/v1/subscription",
    "query": "$.status",
    "authorizationToken": ":apiToken",
    "authorizationType": "Bearer",
    "returnValueTest": {
      "comparator": "==",
      "value": "active"
    }
  }
}
```

`authorizationType` may be `"Bearer"`, `"Basic"`, or `"X-API-Key"`.

## 9. `JsonRpcCondition` — arbitrary JSON-RPC

Call any JSON-RPC service. Here we hit a Solana RPC and require a non-zero SOL balance for a specific, hard-coded Solana account. Hard-coding the account address keeps the access policy fixed at encryption time, which is safer than letting the requester name an arbitrary account via a context variable.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "json-rpc",
    "endpoint": "https://api.mainnet-beta.solana.com",
    "method": "getBalance",
    "params": ["DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263"],
    "query": "$.value",
    "returnValueTest": {
      "comparator": ">",
      "value": 0
    }
  }
}
```

If your access policy genuinely depends on the *requester's* Solana account, you would need to receive that value through an authenticated channel (e.g. a JWT claim validated by a sibling `JwtCondition`) rather than simply letting the requester supply it as a context variable.

## 10. `JsonCondition` — querying a context payload

Unlike `JsonApiCondition` (which fetches), `JsonCondition` evaluates JSON that the decrypter has already supplied. Useful for off-chain attestations.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "json",
    "data": ":attestation",
    "query": "$.claims.kyc_level",
    "returnValueTest": {
      "comparator": ">=",
      "value": 2
    }
  }
}
```

## 11. `JwtCondition` — verify a JWT

Validate a JWT signed by your IDP. The token is supplied at decryption time via the default `:jwtToken` variable.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "jwt",
    "publicKey": "-----BEGIN PUBLIC KEY-----\\nMIIBIjANBgkqhki...\\n-----END PUBLIC KEY-----",
    "expectedIssuer": "https://auth.example.com/"
  }
}
```

`expectedIssuer` is optional but recommended.

`:jwtToken` is the default context-variable name. You can use any name you like by setting `"jwtToken": ":myCustomToken"` — this is useful when a single condition references more than one JWT (e.g. two sibling `JwtCondition`s validating different issuers), since each needs to receive its own value.

## 12. `EcdsaCondition` — verify an ECDSA signature

Verify that a message was signed by a known public key. This is the same pattern the Discord tipping bot uses to prove a Discord interaction is genuine.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "ecdsa",
    "message": ":timestamp:discordPayload",
    "signature": ":signature",
    "verifyingKey": "ED25519_PUBLIC_KEY_HEX",
    "curve": "Ed25519"
  }
}
```

The `"message"` value here is a **concatenation of two context variables**, `:timestamp` and `:discordPayload`. At decryption time the SDK substitutes each `:name` token in sequence, so the string `":timestamp:discordPayload"` becomes `<timestamp_value><payload_value>` — which is what the signer signed.

`:message` and `:signature` are the default context-variable names; like `:jwtToken`, they can be renamed (`"message": ":myMessage"`, `"signature": ":mySig"`) — useful when a single condition verifies multiple distinct signatures.

Supported curves: `SECP256k1`, `NIST256p`, `NIST384p`, `NIST521p`, `Ed25519`, `BRAINPOOLP256r1`.

## 13. `SigningObjectAttributeCondition` — UserOperation field check

Used in TACo Action Control. Validate that an attribute on the object being signed (e.g. an ERC-4337 `UserOperation`) matches an expected value.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "signing-attribute",
    "signingObjectContextVar": ":signingConditionObject",
    "attributeName": "sender",
    "returnValueTest": {
      "comparator": "==",
      "value": "0xEXPECTED_SMART_ACCOUNT_ADDRESS"
    }
  }
}
```

## 14. `SigningObjectAbiAttributeCondition` — validate calldata

Used in TACo Action Control. Decode an attribute (typically `call_data`) using ABI definitions and assert constraints on the decoded parameters. Here: only allow `transfer(recipient, amount)` to a specific address.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "signing-abi-attribute",
    "signingObjectContextVar": ":signingConditionObject",
    "attributeName": "call_data",
    "abiValidation": {
      "allowedAbiCalls": {
        "transfer(address,uint256)": [
          {
            "parameterIndex": 0,
            "returnValueTest": {
              "comparator": "==",
              "value": "0xALLOWED_RECIPIENT"
            }
          },
          {
            "parameterIndex": 1,
            "returnValueTest": {
              "comparator": "<=",
              "value": 1000000
            }
          }
        ]
      }
    }
  }
}
```

For nested calldata validation (e.g. an ERC-4337 `execute()` that wraps a `transfer()`), see the [Discord tipping bot deep-dive](discord-tipping-bot-deep-dive.md).

## 15. `ContextVariableCondition` — assert a custom variable

Apply a return-value test directly to a context variable. Most often used inside a `SequentialCondition` to validate an intermediate result.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "context-variable",
    "contextVariable": ":kycLevel",
    "returnValueTest": {
      "comparator": ">=",
      "value": 2
    }
  }
}
```

## 16. `CompoundCondition` — AND, OR, NOT

Combine up to 5 sub-conditions. Maximum nesting depth is 4. The nesting limit is counted across **all** MultiConditions — `CompoundCondition`, `IfThenElseCondition`, and `SequentialCondition` — not just compounds. A compound that contains an if-then-else that contains a sequential counts as three levels.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "compound",
    "operator": "and",
    "operands": [
      {
        "conditionType": "time",
        "chain": 1,
        "method": "blocktime",
        "returnValueTest": { "comparator": ">", "value": 1735689600 }
      },
      {
        "conditionType": "contract",
        "chain": 1,
        "contractAddress": "0xNFT",
        "standardContractType": "ERC721",
        "method": "balanceOf",
        "parameters": [":userAddress"],
        "returnValueTest": { "comparator": ">", "value": 0 }
      }
    ]
  }
}
```

`"operator": "or"` and `"operator": "not"` work the same way (`not` takes a single operand).

## 17. `SequentialCondition` — derive then check

Two to twenty steps, each binding a `varName` that subsequent steps can reference. Use `operations` on a step to transform the obtained value before storing it.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "sequential",
    "conditionVariables": [
      {
        "varName": "rawBalance",
        "condition": {
          "conditionType": "contract",
          "chain": 8453,
          "contractAddress": "0xUSDC",
          "standardContractType": "ERC20",
          "method": "balanceOf",
          "parameters": [":userAddress"],
          "returnValueTest": { "comparator": ">=", "value": 0 }
        }
      },
      {
        "varName": "checkMinimum",
        "condition": {
          "conditionType": "context-variable",
          "contextVariable": ":rawBalance",
          "returnValueTest": {
            "operations": [
              { "operation": "/=", "value": 1000000 }
            ],
            "comparator": ">=",
            "value": 100
          }
        }
      }
    ]
  }
}
```

## 18. `IfThenElseCondition` — branching

`ifCondition` and `thenCondition` must both be full conditions. Only `elseCondition` accepts a boolean shortcut (`true` means "admit", `false` means "deny").

Before reaching for `IfThenElseCondition`, check whether the rule is really a `CompoundCondition`:

- `if A then true else B` is `OR(A, B)`
- `if A then false else B` is `AND(NOT(A), B)`

`IfThenElseCondition` earns its keep when `thenCondition` and `elseCondition` are themselves meaningful, distinct checks — i.e. holders of the branch-condition's subject face one requirement, and non-holders face another. The example below applies different rules to VIP-pass holders and the general public: holders only need the access window to be open, while everyone else must hold ≥10 USDC.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "if-then-else",
    "ifCondition": {
      "conditionType": "contract",
      "chain": 1,
      "contractAddress": "0xVIP_PASS",
      "standardContractType": "ERC721",
      "method": "balanceOf",
      "parameters": [":userAddress"],
      "returnValueTest": { "comparator": ">", "value": 0 }
    },
    "thenCondition": {
      "conditionType": "time",
      "chain": 1,
      "method": "blocktime",
      "returnValueTest": { "comparator": ">=", "value": 1735689600 }
    },
    "elseCondition": {
      "conditionType": "contract",
      "chain": 1,
      "contractAddress": "0xUSDC",
      "standardContractType": "ERC20",
      "method": "balanceOf",
      "parameters": [":userAddress"],
      "returnValueTest": { "comparator": ">=", "value": 10000000 }
    }
  }
}
```

## 19. Pattern: NFT-or-allowlist gate

Hold the NFT **or** be on a hard-coded allowlist. The allowlist branch uses a `ContextVariableCondition` to check that `:userAddress` is one of the allowed values — no RPC call is needed for a pure allowlist check.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "compound",
    "operator": "or",
    "operands": [
      {
        "conditionType": "contract",
        "chain": 1,
        "contractAddress": "0xNFT",
        "standardContractType": "ERC721",
        "method": "balanceOf",
        "parameters": [":userAddress"],
        "returnValueTest": { "comparator": ">", "value": 0 }
      },
      {
        "conditionType": "context-variable",
        "contextVariable": ":userAddress",
        "returnValueTest": {
          "comparator": "in",
          "value": [
            "0xALLOWED_ADDRESS_1",
            "0xALLOWED_ADDRESS_2",
            "0xALLOWED_ADDRESS_3"
          ]
        }
      }
    ]
  }
}
```

## 20. Pattern: time-windowed paid access

Allow decryption between two timestamps **and** require an active subscription on-chain.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "compound",
    "operator": "and",
    "operands": [
      {
        "conditionType": "time",
        "chain": 8453,
        "method": "blocktime",
        "returnValueTest": { "comparator": ">=", "value": 1735689600 }
      },
      {
        "conditionType": "time",
        "chain": 8453,
        "method": "blocktime",
        "returnValueTest": { "comparator": "<", "value": 1767225600 }
      },
      {
        "conditionType": "contract",
        "chain": 8453,
        "contractAddress": "0xSUBSCRIPTION_REGISTRY",
        "method": "expiresAt",
        "parameters": [":userAddress"],
        "functionAbi": {
          "name": "expiresAt",
          "type": "function",
          "stateMutability": "view",
          "inputs": [{ "name": "user", "type": "address", "internalType": "address" }],
          "outputs": [{ "name": "", "type": "uint256", "internalType": "uint256" }]
        },
        "returnValueTest": { "comparator": ">", "value": 1735689600 }
      }
    ]
  }
}
```

## 21. Pattern: storm-condition open access

A backup safety plan for an outdoor event. If it is raining at the venue **and** the wind is high enough to be unsafe, anyone can decrypt the umbrella-pickup instructions; otherwise only umbrella-NFT holders can. Each branch is a meaningful check — there is no artificial `thenCondition`.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "if-then-else",
    "ifCondition": {
      "conditionType": "json-api",
      "endpoint": "https://api.open-meteo.com/v1/forecast",
      "parameters": { "latitude": 51.5, "longitude": -0.12, "current": "rain" },
      "query": "$.current.rain",
      "returnValueTest": { "comparator": ">", "value": 0 }
    },
    "thenCondition": {
      "conditionType": "json-api",
      "endpoint": "https://api.open-meteo.com/v1/forecast",
      "parameters": { "latitude": 51.5, "longitude": -0.12, "current": "wind_speed_10m" },
      "query": "$.current.wind_speed_10m",
      "returnValueTest": { "comparator": ">", "value": 30 }
    },
    "elseCondition": {
      "conditionType": "contract",
      "chain": 1,
      "contractAddress": "0xUMBRELLA_NFT",
      "standardContractType": "ERC721",
      "method": "balanceOf",
      "parameters": [":userAddress"],
      "returnValueTest": { "comparator": ">", "value": 0 }
    }
  }
}
```

## 22. Pattern: balance fetched, normalised, then asserted

A `SequentialCondition` that fetches a wei balance, divides it down to whole ETH using `operations`, and then asserts a minimum.

```json
{
  "version": "1.0.0",
  "condition": {
    "conditionType": "sequential",
    "conditionVariables": [
      {
        "varName": "weiBalance",
        "condition": {
          "conditionType": "rpc",
          "chain": 1,
          "method": "eth_getBalance",
          "parameters": [":userAddress", "latest"],
          "returnValueTest": { "comparator": ">=", "value": 0 }
        },
        "operations": [
          { "operation": "weiToEth" }
        ]
      },
      {
        "varName": "checkWholeEth",
        "condition": {
          "conditionType": "context-variable",
          "contextVariable": ":weiBalance",
          "returnValueTest": { "comparator": ">=", "value": 5 }
        }
      }
    ]
  }
}
```

## Available `operations`

Operations can be applied to obtained values inside `returnValueTest.operations` or `ConditionVariable.operations`. Up to five per array.

| Operation | Purpose |
| :--- | :--- |
| `+=`, `-=`, `*=`, `/=`, `%=` | Arithmetic with the supplied `value` |
| `toTokenBaseUnits` | Multiply by `10^value` (decimals → base units) |
| `weiToEth`, `ethToWei` | Native unit conversions |
| `int`, `float`, `bool`, `str` | Type coercion |
| `round`, `floor`, `ceil`, `abs` | Numeric rounding |
| `len`, `min`, `max`, `sum`, `avg` | Aggregations on arrays |
| `index` | Pick an array element |
| `keccak` | Hash a string |
| `create2` | Compute a CREATE2 address locally given `deployerAddress` + `bytecodeHash` |

For the full enum see [VariableOperation in the schema reference](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md#variableoperation).

---

Need something this cookbook does not cover? Read the [schema reference](https://github.com/nucypher/taco-web/blob/signing-epic/packages/taco/schema-docs/condition-schemas.md), then ask an LLM with the [building-with-llms workflow](building-with-llms.md).
