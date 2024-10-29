# JsonAPICondition

The `JsonApiCondition` is useful when access control decisions need to be based on data retrieved from an external HTTPS API that returns data in JSON format. It can be leveraged to integrate external off-chain data into your access control policies, ensuring access decisions are based on dynamic, real-time data retrieved from JSON APIs.

Potential use cases include geographic restrictions based on IP address or weather-based access control.

The condition works by sending an HTTPS GET request to a specified JSON API endpoint, extracting relevant data from the response using a [`JSONPath`](https://goessner.net/articles/JsonPath/) query, and then comparing the extracted value against an expected result.

It is composed of the following properties:

* `endpoint`: the HTTPS URI for the JSON API endpoint that will be queried, e.g.`https://api.example.com/user/status`
* `parameters`_(Optional)_: a key-value mapping of parameter names and values to pass as part of the HTTPS GET request. These parameters will be appended to the URL as query string parameters.
* `query`_(Optional)_: a `JSONPath` query used to extract specific data from the JSON response.
* [`authorizationToken`](jsonapicondition.md#authorization-token) (Optional): A bearer token that will be included in the HTTPS `Authorization` header. It enables the use of endpoints that require OAuth/JWT authorization.&#x20;
* [`returnValueTest`](../#returnvaluetest): the test to validate the value extracted by the JSONPath query.&#x20;

**Error Handling**

* If the HTTPS response does not return a status code of `200`, the condition will fail automatically, and access will be denied.
* If the `JSONPath` query is provided but cannot properly extract the desired value, the condition will fail, resulting in access being denied.
* If an invalid `authorizationToken` is provided, the call to the API will fail, causing the condition to fail and access to be denied.

## Authorization

Some API endpoints require bearer tokens to verify authorization. Because an authorization token is specific to the requester, the `authorizationToken` value must always be provided as a [user-defined custom context variable](../../authentication/conditioncontext-and-context-variables.md#context-variables). The application should provide this value at decryption time via the [Condition Context](../../authentication/conditioncontext-and-context-variables.md).

Tokens must be provided as context variables for the following reasons:

* **Security:** Hardcoding the token would expose it in the plaintext condition associated with the encrypted data.
* **Flexibility:** Hardcoding ties the token to a specific user rather than the requester.
* **Expiry:** Tokens typically expire, so hard coding would lead to eventual invalidation.

For these reasons, only custom context variables are allowed for the `authorizationToken` property.

## Example

```typescript
import { conditions } from '@nucypher/taco';

const firstBookPrice = new conditions.base.jsonApi.JsonApiCondition({
  endpoint: 'https://api.books.com/data',
  parameters: {
    'name': 'It'
  },
  query: '$.store.book[0].price',
  authorizationToken: ":authToken",
  returnValueTest: {
    comparator: "==",
    value: 1
  },
});
```

The condition would be satisfied if the API endpoint returned something analogous to the following:

<pre class="language-json"><code class="lang-json"><strong>{  
</strong>  "store":{
    "book":[
      {
        "name": "It",
        "author": "Stephen King",
        "price": 1
      }
    ]
  }
}
</code></pre>

## Development References

* Client-side:
  * [https://github.com/nucypher/taco-web/pull/550](https://github.com/nucypher/taco-web/pull/550)
  * [https://github.com/nucypher/taco-web/pull/561](https://github.com/nucypher/taco-web/pull/561)
  * [https://github.com/nucypher/taco-web/pull/599](https://github.com/nucypher/taco-web/pull/599)
* Server-side:&#x20;
  * [https://github.com/nucypher/nucypher/pull/3511](https://github.com/nucypher/nucypher/pull/3511)
  * [https://github.com/nucypher/nucypher/pull/3560](https://github.com/nucypher/nucypher/pull/3560)
