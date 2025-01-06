# JSON Endpoint Conditions

JSON conditions are useful when access control decisions need to be based on data retrieved from an external HTTPS JSON endpoint ([API](jsonapicondition.md) or [RPC](jsonrpccondition.md)) that returns data in JSON format. They can be leveraged to integrate external off-chain data into your access control policies, ensuring access decisions are based on dynamic, real-time data retrieved from JSON endpoints.

## Authorization

Some JSON endpoints require bearer tokens to verify authorization. Because an authorization token is specific to the requester, the `authorizationToken` value must always be provided as a [user-defined custom context variable](../../../authentication/conditioncontext-and-context-variables.md#context-variables). The application should provide this value at decryption time via the [Condition Context](../../../authentication/conditioncontext-and-context-variables.md).

Where applicable, authorization tokens must be provided as context variables for the following reasons:

* **Security:** Hardcoding the token would expose it in the plaintext condition associated with the encrypted data.
* **Flexibility:** Hardcoding ties the token to a specific user rather than the requester.
* **Expiry:** Tokens typically expire, so hard coding would lead to eventual invalidation.

For these reasons, only custom context variables are allowed for the optional `authorizationToken` property.

## **Special Considerations**

* **Immutable Endpoint URLs:**\
  JSON conditions rely on hardcoded HTTPS endpoints as part of the condition definition. Once a condition is associated with a piece of ciphertext, the endpoint URL becomes fixed and cannot be modified. This immutability ensures the integrity of the condition but introduces challenges if the endpoint URL becomes unavailable, deprecated, or requires updates.
* **Dependency on External Systems:**\
  Since JSON conditions depend on external HTTPS endpoints, the availability and reliability of these endpoints directly impact the functionality of the condition. If the endpoint experiences downtime, latency issues, or is retired, access control decisions relying on it may fail.
* **Versioning and Compatibility:**\
  Endpoint URLs often correspond to specific API versions or specifications. If the endpoint provider introduces breaking changes, conditions that depend on the outdated version may no longer function correctly.&#x20;
* **Security Risks:**\
  Hardcoding the endpoint URL makes the system reliant on the endpoint's continued security and trustworthiness. If the endpoint is compromised or ownership changes, the system could be exposed to potential risks.

The above may not be a concern for some situations, so it is important to evaluate your use case carefully.
