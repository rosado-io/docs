# Authenticate With JWT

In this section, we will go through how to decode the JWT token to obtain the currently logged-in user.

Before we start, make sure you selected the option "**Issue JWT as access token**" in your Application settings in Portal.

## Decode user from an access token

Use your Authgear endpoint as `baseAddress`

{% tabs %}
{% tab title="Go" %}
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
    baseAddress       = "https://mycompany.authgearapps.com"
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

{% tab title="Python" %}
Install packages

```bash
pip install cryptography
pip install PyJWT
```

Example code

```python
import json
from contextlib import closing
from urllib.request import urlopen

import jwt
from jwt import PyJWKClient

base_address = "https://mycompany.authgearapps.com"


def fetch_jwks_uri(base_address):
    doc_url = base_address + "/.well-known/openid-configuration"
    with closing(urlopen(doc_url)) as f:
        doc = json.load(f)
    jwks_uri = doc["jwks_uri"]
    if not jwks_uri:
        raise Exception('Failed to fetch jwks uri.')
    return jwks_uri


def parse_header(authz_header):
    parts = authz_header.split(" ")
    if len(parts) != 2:
        return

    scheme = parts[0]
    if scheme.lower() != "bearer":
        return

    return parts[1]


authz_header = headers.get("Authorization")
# get jwt token from Authorization header
token = parse_header(authz_header)
if token:
    # fetch jwks_uri from Authgear
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

```
{% endtab %}
{% endtabs %}

## Decode user from cookies

{% hint style="info" %}
JWT in cookies is not supported yet [which you can track here](https://github.com/authgear/authgear-server/issues/1180).
{% endhint %}

