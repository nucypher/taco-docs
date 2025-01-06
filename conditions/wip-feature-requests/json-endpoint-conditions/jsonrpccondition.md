# JsonRpcCondition

The `JsonRpcCondition` is designed for access control decisions that rely on data from an external HTTPS JSON RPC endpoint following the [JSON RPC 2.0 specification](https://www.jsonrpc.org/specification).&#x20;

The condition operates by sending an HTTPS `POST` request to a specified JSON RPC endpoint using a [`JSONPath`](https://goessner.net/articles/JsonPath/) query to extract relevant data from the response and comparing the extracted value to an expected result.

Potential use cases include utilizing JSON RPC 2.0 endpoints, including sending JSON RPC requests to non-EVM blockchain RPC endpoints, e.g. getting the block time from Bitcoin or Solana. A separate non-EVM blockchain condition may be added if there is sufficient use in this area.

It is composed of the following properties:

* `endpoint`: the HTTPS URI for the JSON RPC endpoint that will be queried, e.g.`https://api.example.com`
* `method`: the JSON RPC method to be invoked
* `params`_(Optional)_: Parameters for the specified method, provided as either a dictionary or an array.
* `query`_(Optional)_: a `JSONPath` query used to extract specific data from the JSON response. The query is relative to the `result` entry included in the JSON response.
* [`authorizationToken`](jsonrpccondition.md#authorization-token) _(Optional)_: A bearer token that will be included in the HTTPS `Authorization` header. It enables the use of endpoints that require OAuth/JWT authorization.&#x20;
* [`returnValueTest`](../../#returnvaluetest): the test to validate the value extracted by the JSONPath query.&#x20;

**Error Handling**

* If the HTTPS response does not return a status code of `200`, the condition will fail automatically, and access will be denied.
* If the `JSONPath` query is provided but cannot properly extract the desired value, the condition will fail, resulting in access being denied.
* If an invalid `authorizationToken` is provided, the call to the API will fail, causing the condition to fail and access to be denied.

## Example

```typescript
import { conditions } from '@nucypher/taco';

const firstBookPrice = new conditions.base.jsonRpc.JsonRpcCondition({
  endpoint: 'https://math.example.com/',
  method: 'subtract',
  params: [42, 23],
  query: '$.mathresult',
  authorizationToken: ":authToken",
  returnValueTest: {
    comparator: "==",
    value: 19
  },
});
```

The JSON data for the HTTP POST request to the RPC endpoint will look like the following:

```json
{
    "jsonrpc": "2.0",
    "id":1,
    "method": "subtract",
    "params": [
        42,
        23
    ]
}
```

The condition would be satisfied if the JSON RPC endpoint returned something analogous to the following:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "mathresult": 19
    },
    "id": 1
}
```

## Development References

* Client-side:
  * [https://github.com/nucypher/taco-web/pull/606](https://github.com/nucypher/taco-web/pull/606)
* Server-side:&#x20;
  * [https://github.com/nucypher/nucypher/pull/3571](https://github.com/nucypher/nucypher/pull/3571)
