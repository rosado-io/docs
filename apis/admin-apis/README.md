# Admin APIs

The Admin API allows your server to manage users via a GraphQL endpoint. You can list users, search users, view user details, and many more. In fact, the user management part of the portal is built with the Admin API.

## The Admin API GraphQL endpoint

The Admin API GraphQL endpoint is at `/_api/admin/graphql`. For example, if your app is `myapp` , then the endpoint is `https://myapp.authgearapps.com/_api/admin/graphql` .

## Accessing the Admin API GraphQL endpoint

Accessing the Admin API GraphQL endpoint requires your server to generate a valid JWT and include it as `Authorization` HTTP header.

### Obtaining the private key for signing JWT

* Go to **Settings** -> **Admin API**
* Click **Download** to download the private key. Make it available to your server.
* **Copy** the key ID. Make it available to your server.

### Generating the JWT with the private key

Here is the sample code of how to generate the JWT with the private key.

{% tabs %}
{% tab title="Go" %}
```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "time"

    "github.com/lestrrat-go/jwx/jwa"
    "github.com/lestrrat-go/jwx/jwk"
    "github.com/lestrrat-go/jwx/jws"
    "github.com/lestrrat-go/jwx/jwt"
)

// Replace "myapp" with your app ID here.
const AppID = "myapp"

// Replace "mykid" with the key ID you see in the portal.
const KeyID = "mykid"

func main() {
    // Replace the following call with your own way to get the private key.
    f, err := os.Open("private-key.pem")
    if err != nil {
        panic(err)
    }
    defer f.Close()
    jwkSet, err := jwk.ParseReader(f, jwk.WithPEM(true))
    if err != nil {
        panic(err)
    }
    key, _ := jwkSet.Get(0)

    now := time.Now().UTC()
    payload := jwt.New()
    _ = payload.Set(jwt.AudienceKey, AppID)
    _ = payload.Set(jwt.IssuedAtKey, now.Unix())
    _ = payload.Set(jwt.ExpirationKey, now.Add(5*time.Minute).Unix())

    // The alg MUST be RS256.
    alg := jwa.RS256
    hdr := jws.NewHeaders()
    hdr.Set("typ", "JWT")
    hdr.Set("alg", alg.String())
    hdr.Set("kid", KeyID)

    buf, err := json.Marshal(payload)
    if err != nil {
        panic(err)
    }

    token, err := jws.Sign(buf, alg, key, jws.WithHeaders(hdr))
    if err != nil {
        panic(err)
    }

    fmt.Printf("%v\n", string(token))
}
```
{% endtab %}

{% tab title="Python" %}
```python
import jwt
from datetime import datetime, timedelta

private_key = open("private-key.pem", "r").read()

APP_ID = "myapp"
KID = "mykid"

now = datetime.now()

payload = {
    "aud": [APP_ID],
    "iat": int(now.timestamp()),
    "exp": int((now + timedelta(minutes=5)).timestamp()),
}

token = jwt.encode(
    payload,
    private_key,
    "RS256",
    headers={
        "kid": KID,
    },
)

print(token)
```
{% endtab %}
{% endtabs %}

### Including the JWT in the HTTP request

After generating the JWT, you must include it in **EVERY** request you send to the Admin API endpoint. Here is how it looks like

```
Authorization: Bearer <JWT>
```

The header is the standard Authorization HTTP header. The token type **MUST** be Bearer.

### Optional: Caching the JWT

As you can see in the sample code, you expiration time of the JWT is 5 minutes. You make it last longer and cache it to avoid generating it on every request.

## Trying out the Admin API GraphQL endpoint

{% hint style="danger" %}
The GraphiQL tool is NOT a sandbox environment and all changes will be made on real data. Use with care!
{% endhint %}

The above instruction is for server-side integration. If you want to explore what it can do, you can visit the GraphiQL tool.

* Go to **Settings** -> **Admin API**
* Click on the **GraphiQL tool** link

### Inspecting the GraphQL schema

In the GraphiQL tool, you can toggle the schema documentation by pressing the Docs button in the top right corner.
