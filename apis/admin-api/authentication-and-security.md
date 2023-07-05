---
description: >-
  The Admin API is protected by cryptographic keys. Learn how to generate a
  valid JWT to authorize your request in this article.
---

# Authentication and Security

## Accessing the Admin API GraphQL endpoint

Accessing the Admin API GraphQL endpoint requires your server to generate a valid JWT and include it as `Authorization` HTTP header.

## Obtaining the private key for signing JWT

* Go to **Settings** -> **Admin API**
* Click **Download** to download the private key. Make it available to your server.
* **Copy** the key ID. Make it available to your server.

## Generating the JWT with the private key

#### Sample code to generate the JWT

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

// Replace "myapp" with your project ID here. 
// It is the first part of your Authgear endpoint. 
// e.g. The project ID is "myapp" for "https://myapp.authgearapps.com"
const ProjectID = "myapp"

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
    _ = payload.Set(jwt.AudienceKey, ProjectID)
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

# Replace "myapp" with your project ID here. 
# It is the first part of your Authgear endpoint. 
# e.g. The project ID is "myapp" for "https://myapp.authgearapps.com"
PROJECT_ID = "myapp"

# Replace "mykid" with the key ID you see in the portal.
KEY_ID = "mykid"

now = datetime.now()

payload = {
    "aud": [PROJECT_ID],
    "iat": int(now.timestamp()),
    "exp": int((now + timedelta(minutes=5)).timestamp()),
}

token = jwt.encode(
    payload,
    private_key,
    "RS256",
    headers={
        "kid": KEY_ID,
    },
)

print(token)
```
{% endtab %}
{% endtabs %}

#### Example of the JWT header

```json
{
  "alg": "RS256",
  "kid": "REPLACE_YOUR_KEY_ID_HERE",
  "typ": "JWT"
}
```

#### Example of the JWT payload

```json
{
  "aud": [
    "REPLACE_YOUR_PROJECT_ID_HERE"
  ],
  "exp": 1136257445,
  "iat": 1136171045
}
```

## Including the JWT in the HTTP request

After generating the JWT, you must include it in **EVERY** request you send to the Admin API endpoint. Here is how it looks like

```
Authorization: Bearer <JWT>
```

The header is the standard Authorization HTTP header. The token type **MUST** be Bearer.

## Optional: Caching the JWT

As you can see in the sample code, you expiration time of the JWT is 5 minutes. You make it last longer and cache it to avoid generating it on every request.

## Admin API Key rotation

You should regularly change the API key used to authenticate API requests. It enhances security by minimizing the impact of compromised keys.

To rotate the API key

1. Go to **Portal** > **Advanced** > **Admin API**
2. Under "List of Admin API keys", click "Generate new key pair"
3. At this point both keys can be used to authenticate the admin API requests.
4. Make sure all your systems is updated to use the new key
5. Delete the old API key
