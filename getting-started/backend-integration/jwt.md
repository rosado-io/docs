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

// DecodeUser parse request Authorization header and obtain user id and claims
func DecodeUser(r *http.Request) (string, map[string]interface{}, error) {
	// fetch jwks from server
  // you can cache the value of jwks to have better performance
	set, err := jwk.Fetch(context.Background(), baseAddress+"/oauth2/jwks")
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

{% tab title="Second Tab" %}

{% endtab %}
{% endtabs %}

## Decode user from cookies

{% hint style="info" %}
JWT in cookies is not supported yet [which you can track here](https://github.com/authgear/authgear-server/issues/1180).
{% endhint %}

