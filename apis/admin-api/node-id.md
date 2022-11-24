# Generate the node id for Admin GraphQL query and mutation

The node id is a globally unique identifier of an object. It is needed when you call the GraphQL query and mutation. In this section, we will go through how to generate the node ID for calling the Admin GraphQL API.

The node id is an [base64url](https://datatracker.ietf.org/doc/html/rfc4648#section-5) encoded string with format of `<NODE_TYPE>:<ID>`.

## Generate the user node id example

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
	"encoding/base64"
	"fmt"
)

func main() {
	nodeID := base64.RawURLEncoding.EncodeToString([]byte("User:97b1c929-842c-415c-a7df-6967efdda160"))
	fmt.Println(nodeID)
}
```
{% endtab %}
{% tab title="Python" %}
```python
import base64

base64.urlsafe_b64encode(b'User:97b1c929-842c-415c-a7df-6967efdda160').replace(b'=', b'')
```
{% endtab %}
{% endtabs %}

## Fetch the user object example

The query:

```graphql
query ($userID: ID!) {
  node(id: $userID) {
    ... on User {
      id
      standardAttributes
    }
  }
}
```

The variables:

```json
{
  "userID": "<BASE64URL_ENCODED_USER_NODE_ID>"
}
```
