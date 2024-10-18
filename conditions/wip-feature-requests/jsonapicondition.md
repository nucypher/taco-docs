# JsonAPICondition

The `JsonApiCondition` is useful when access control decisions need to be based on data retrieved from an external HTTPS API that returns data in JSON format. It can be leveraged to integrate external off-chain data into your access control policies, ensuring access decisions are based on dynamic, real-time data retrieved from JSON APIs.

Potential use cases include geographic restrictions based on IP address or weather-based access control.

The condition works by sending an HTTPS GET request to a specified JSON API endpoint, extracting relevant data from the response using a [`JSONPath`](https://goessner.net/articles/JsonPath/) query, and then comparing the extracted value against an expected result.

It is composed of the following properties:

* `endpoint`: the HTTPS URI for the JSON API endpoint that will be queried, e.g.`https://api.example.com/user/status`
* `parameters`_(Optional)_: a key-value mapping of parameter names and values to pass as part of the HTTPS GET request. These parameters will be appended to the URL as query string parameters.
* `query`: a `JSONPath` query used to extract specific data from the JSON response.&#x20;
* [`returnValueTest`](../#returnvaluetest): the test to validate the value extracted by the JSONPath query.&#x20;

**Error Handling**

* If the HTTPS response does not return a status code of `200`, the condition will fail automatically, and access will be denied.
* If the `JSONPath` query cannot extract the desired value, the condition will fail, resulting in access being denied.

## Example

```typescript
import { conditions } from '@nucypher/taco';

const firstBookPrice = new conditions.base.jsonApi.JsonApiCondition({
  endpoint: 'https://api.books.com/data',
  parameters: {
    'name': 'It'
  },
  query: '$.store.book[0].price',
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
* Server-side: [https://github.com/nucypher/nucypher/pull/3511](https://github.com/nucypher/nucypher/pull/3511)
