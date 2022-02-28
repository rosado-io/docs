---
description: >-
  Authenticate the incoming HTTP requests by validating JWT in your application
  server
---

# Validate JWT in your application server

In this section, we will go through how to decode the JWT token to obtain the currently logged-in user.

Before we start, make sure the option **Issue JWT as access token** is enabled in your Application settings in the Portal.&#x20;

![Make sure "Issue JWT as access token" is enabled in your application](../../.gitbook/assets/issue-jwt-as-access-token.png)

## Find the JSON Web Key Sets (JWKS) endpoint

This Discovery endpoint serves a JSON document containing the OpenID Connect configuration of your app. It includes the authorization endpoint, the token endpoint and the JWKS endpoint.

`https://<YOUR_AUTHGEAR_ENDPOINT>/.well-known/openid-configuration`&#x20;

Here is [an example of how it looks](https://accounts.portal.authgearapps.com/.well-known/openid-configuration).

The JSON Web Key Sets (JWKS) endpoint can be found in `jwk_url` in the configuration.

**OpenID Connect Configuration JSON**

```json
{
    "jwks_uri": "..."
    ...
}
```

## Decode user from an access token

Follow this step-by-step example to verify and decode the JWT token.

{% tabs %}
{% tab title="Python" %}
### Step 1: Install packages

```bash
pip install cryptography
pip install PyJWT
```

### Step 2: Find the JSON Web Key Sets (JWKS) endpoint

Define a function to find the JWKS endpoint from the OpenID Connect configuration. Use your Authgear endpoint as `base_address`

```python
import json
from contextlib import closing
from urllib.request import urlopen

base_address = "https://<your_app_endpoint>"

def fetch_jwks_uri(base_address):
    doc_url = base_address + "/.well-known/openid-configuration"
    with closing(urlopen(doc_url)) as f:
        doc = json.load(f)
    jwks_uri = doc["jwks_uri"]
    if not jwks_uri:
        raise Exception('Failed to fetch jwks uri.')
    return jwks_uri
```

### Step 3: Get JWT token from the Authorization header

Define a function to extract the access token from the Authorization header in the incoming request. It should look like `Authorization: Bearer <access_token>`.

```python
def parse_header(authz_header):
    parts = authz_header.split(" ")
    if len(parts) != 2:
        return

    scheme = parts[0]
    if scheme.lower() != "bearer":
        return

    return parts[1]
```

### Step 4: Verify and decode the JWT token

Here show an example of using Flask web framework to guard a path. You may need to adjust some of the codes to suit your technologies.

```python
from flask import request
import jwt
from jwt import PyJWKClient

@app.route("/hello")
def hello():
    authz_header = request.headers.get("Authorization")
    if not authz_header:
        return {
            "message": "authz header not found"
        }

    # get jwt token from Authorization header
    token = parse_header(authz_header)
    if token:
        try:
            # fetch jwks_uri from the Authgear Discovery Endpoint
            jwks_uri = fetch_jwks_uri(base_address)
            # Reuse PyJWKClient for better performance
            jwks_client = PyJWKClient(jwks_uri)
            signing_key = jwks_client.get_signing_key_from_jwt(token)
            user_data = jwt.decode(
                token,
                signing_key.key,
                algorithms=["RS256"],
                audience=base_address,
                options={"verify_exp": True},
            )
            return {
                "message": "Hello!",
                "user_data": user_data
            }
        except:
            return {
                "message": "JWT decode failed"
            }
    else:
        return {
            "message": "no token"
        }
```
{% endtab %}

{% tab title="Go" %}
Use your Authgear endpoint as `base_address`

```go
import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "regexp"
    "time"

    "github.com/lestrrat-go/jwx/jwk"
    "github.com/lestrrat-go/jwx/jwt"
)


var (
    authzHeaderRegexp = regexp.MustCompile("(?i)^Bearer (.*)$")
    baseAddress       = "https://<your_app_endpoint>"
)

type OIDCDiscoveryDocument struct {
    JWKSURI string `json:"jwks_uri"`
}

func FetchOIDCDiscoveryDocument(endpoint string) (*OIDCDiscoveryDocument, error) {
    resp, err := http.DefaultClient.Get(endpoint)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf(
            "failed to fetch discovery document: unexpected status code: %d",
            resp.StatusCode,
        )
    }

    var document OIDCDiscoveryDocument
    err = json.NewDecoder(resp.Body).Decode(&document)
    if err != nil {
        return nil, err
    }
    return &document, nil
}

func FetchJWK(baseAddress string) (jwk.Set, error) {
    doc, err := FetchOIDCDiscoveryDocument(
        baseAddress + "/.well-known/openid-configuration",
    )
    if err != nil {
        return nil, err
    }

    set, err := jwk.Fetch(context.Background(), doc.JWKSURI)
    return set, err
}

// DecodeUser parse request Authorization header and obtain user id and claims
func DecodeUser(r *http.Request) (string, map[string]interface{}, error) {
    // fetch jwks_uri from Authgear
    // you can cache the value of jwks to have better performance
    set, err := FetchJWK(baseAddress)
    if err != nil {
        return "", nil, fmt.Errorf("failed to fetch JWK: %s", err)
    }

    // get jwt token from Authorization header
    authzHeader := r.Header.Get("Authorization")
    match := authzHeaderRegexp.FindStringSubmatch(authzHeader)
    if len(match) != 2 {
        return "", nil, fmt.Errorf("no token")
    }

    // parse jwt token
    token, err := jwt.ParseString(match[1], jwt.WithKeySet(set))
    if err != nil {
        return "", nil, fmt.Errorf("invalid token: %s", err)
    }

    // validate jwt token
    err = jwt.Validate(token,
        jwt.WithClock(jwt.ClockFunc(
            func() time.Time { return time.Now().UTC() },
        )),
        jwt.WithAudience(baseAddress),
    )
    if err != nil {
        return "", nil, fmt.Errorf("invalid token: %s", err)
    }

    return token.Subject(), token.PrivateClaims(), nil
}

func handler(w http.ResponseWriter, r *http.Request) {
    // decode user example
    userid, claims, err := DecodeUser(r)
    isUserVerified, _ :=
        claims["https://authgear.com/claims/user/is_verified"].(bool)
    isAnonymousUser, _ :=
        claims["https://authgear.com/claims/user/is_anonymous"].(bool)

    // ... your handler logic
}
```
{% endtab %}

{% tab title="Node.js" %}
{% hint style="success" %}
TODO: Node.js example is coming soon
{% endhint %}

Here is an example demonstrates how to add authorization to an Express.js API

### Step 1: Install dependencies

```bash
npm install --save express-jwt jwks-rsa
```

### Step 2: Configure the middleware

### Step 3: Protect API Endpoints
{% endtab %}
{% endtabs %}

## Decode user from cookies

Validating JWT in your application server is _currently_ only available for **Token-based authentication**.

{% hint style="info" %}
For Cookie-based authentication, JWT in cookies is not supported yet. [You can track the issue here](https://github.com/authgear/authgear-server/issues/1180).
{% endhint %}
