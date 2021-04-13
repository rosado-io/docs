---
description: How to make authorized request to your application server after login with Authgear
---

# Using Authgear SDK to call your application server

In this section, we are going to explain how to setup Authorization header to your application request by using Authgear SDK. If you are using session cookies in your web application, you can skip this section as session cookies are handled by browser automatically.

### iOS

- Configure Authgear SDK

    ```swift
    let authgear = Authgear(clientId: clientId, endpoint: endpoint)
    authgear.configure() { result in
        switch result {
        case .success():
            // configured successfully
        case let .failure(error):
            // failed to configured
        }
    }
    ```

- Determine if user logged in by checking the session state, the session state only reflect the SDK local state. That means even `sessionState` is `authenticated`, the session maybe invalid if it is revoked remotely. You will only know that after calling the server.

    ```swift
    // value can be .noSession or .authenticated
    authgear.sessionState
    ```

- Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will refresh the access token only if it has expired. 

    ```swift 
    authgear.refreshAccessTokenIfNeeded() { result in
        switch result {
        case .success():
            // access token is ready to use
            // if user is not logged in, access token will be empty
            // if user is logged in, include access token into the Authorization header.
            if let accessToken = authgear.accessToken {
                // example only, you can use your own networking library
                var urlRequest = URLRequest(url: "YOUR_SERVER_URL")
                urlRequest.setValue("Bearer \(accessToken)", forHTTPHeaderField: "authorization")
                // ... continue making your request
            }
        case let .failure(error):
            // failed to refresh access token, the refresh token maybe expired or revoked
        }
    }
    ```

- Handle the case that the session is revoked after SDK obtained the access token. In this case the app will call your application server with access token. But you will find that the access token is invalid, after checking with Authgear in your application server. You may want to logout the user in the SDK in this scenario, you can call force logout.

    ```swift
    authgear.logout(force: true) { result in
        switch result {
        case .success():
            // force logout successfully
            // authgear.sessionState becomes .noSession
        case let .failure(error):
            // failed to force logout
        }
    }

    // example
    // if your application server return HTTP status code 401 for unauthorized request
    if let response = response as? HTTPURLResponse {
        if response.statusCode == 401 {
            authgear.logout(force: true)
        }
    }
    ```

### Android


### Javascript


