---
description: How to make authorized request to your application server after login with Authgear
---

# Using Authgear SDK to call your application server

In this section, we are going to explain how to make authorized request to your application server by using Authgear SDK. If you are using session cookies in your web application, you can skip this section as session cookies are handled by browser automatically.


# Setup Overview

1. To determine which user is calling your server, you will need to include Authorization header in every request that send to your application sever.
1. You will need to setup [Backend integration](../how-tos/backend-integration.md), Authgear will help you to resolve the Authorization header to determine whether the incoming HTTP request is authenticated or not.

In the below section, we will explain how to setup SDK to include Authorization header to your application requests.

# SDK setup

- Configure the Authgear SDK with the Authgear endpoint and client id. The SDK must be properly configured before using, you can call `configure` multiple times but you should only need to call it once. No network call will be triggered during `configure`.

    {% tabs %}
    {% tab title="Javascript" %}
    ```javascript
    authgear
        .configure({
            clientID: "a_random_generated_string",
            endpoint: "http://localhost:3000",
        })
        .then(() => {
            // configured successfully
        })
        .catch((e) => {
            // failed to configured
        });
    ```
    {% endtab %}
    {% tab title="iOS" %}
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
    {% endtab %}
    {% tab title="Android" %}
    ```java
    ConfigureOptions configureOptions = new ConfigureOptions();
    Authgear authgear = new Authgear(getApplication(), clientID, endpoint, null);
    authgear.configure(configureOptions, new OnConfigureListener() {
        @Override
        public void onConfigured() {
            // configured successfully
        }

        @Override
        public void onConfigurationFailed(@NonNull Throwable throwable) {
            // failed to configured
        }
    });
    ```
    {% endtab %}
    {% endtabs %}

- `sessionState` reflect the user logged in state. Right after configure, the session state only reflect the SDK local state. That means even `sessionState` is `AUTHENTICATED`, the session maybe invalid if it is revoked remotely. You will only know that after calling the server, call `fetchUserInfo` as soon as it is proper to do so, e.g. when the device goes online.

    {% tabs %}
    {% tab title="Javascript" %}
    ```javascript
    // value can be NO_SESSION or AUTHENTICATED
    let sessionState = authgear.sessionState;
    
    if (sessionState === "AUTHENTICATED") {
        authgear
            .fetchUserInfo()
            .then((userInfo) => {
                // sessionState is now up to date
                // read the userInfo if needed
            })
            .catch((e) => {
                // sessionState is now up to date, it will change to NO_SESSION if the session is invalid
            });
    }
    ```
    {% endtab %}
    {% tab title="iOS" %}
    ```swift
    // value can be .noSession or .authenticated. After authgear.configure, it only reflect SDK local state.
    let sessionState = authgear.sessionState

    // call fetchUserInfo to see if the session is valid
    if authgear.sessionState == .authenticated {
        authgear.fetchUserInfo { userInfoResult in
            // sessionState is now up to date, it will change to .noSession if the session is invalid
            let sessionState = authgear.sessionState

            switch userInfoResult {
            case let .success(userInfo):
                // read the userInfo if needed
            case let .failure(error):
                // failed to fetch user info, the refresh token maybe expired or revoked
        }
    }
    ```
    {% endtab %}
    {% tab title="Android" %}
    ```java
    // value can be NO_SESSION or AUTHENTICATED
    SessionState sessionState = authgear.getSessionState();
    
    if (sessionState == SessionState.AUTHENTICATED) {
        authgear.fetchUserInfo(new OnFetchUserInfoListener() {
            @Override
            public void onFetchedUserInfo(@NonNull UserInfo userInfo) {
                // sessionState is now up to date
                // read the userInfo if needed
            }

            @Override
            public void onFetchingUserInfoFailed(@NonNull Throwable throwable) {
                // sessionState is now up to date, it will change to NO_SESSION if the session is invalid
            }
        });
    }
    ```
    {% endtab %}
    {% endtabs %}

- **Javascript Only**. Authgear SDK provides `fetch` function for you to call your application server. The `fetch` function will include Authorization header in your application request, and handle refresh access token automatically. `authgear.fetch` implement [fetch](https://fetch.spec.whatwg.org/). If you are using another network library and want to include Authorization header yourself. You can skip this and go to next step.

    ```javascript
    authgear
        .fetch("YOUR_SERVER_URL")
        .then(response => response.json())
        .then(data => console.log(data));
    ```

- You will need to include Authorization header in your application request, you can skip this if you are using `authgear.fetch` in Javascript SDK. Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will check and make network call only if access token has expired. Include the access token into Authorization header of your application request.

    {% tabs %}
    {% tab title="Javascript" %}
    ```java
    authgear
        .refreshAccessTokenIfNeeded()
        .then(() => {
            // access token is ready to use
            // accessToken can be string or undefined
            // accessToken will be empty if user is not logged in or session is invalid
            const accessToken = authgear.accessToken;

            // include Authorization header in your application request
            const headers = {
                Authorization: `bearer ${accessToken}`
            };
        });
    ```
    {% endtab %}
    {% tab title="iOS" %}
    ```swift 
    authgear.refreshAccessTokenIfNeeded() { result in
        switch result {
        case .success():
            // access token is ready to use
            // if user is not logged in or session is invalid, access token will be empty
            // if user is logged in, include access token into the Authorization header
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
    {% endtab %}
    {% tab title="Android" %}
    ```java
    authgear.refreshAccessTokenIfNeeded(new OnRefreshAccessTokenIfNeededListener() {
        @Override
        public void onFinished() {
            // access token is ready to use
            // if user is not logged in or session is invalid, access token will be empty
            // if user is logged in, include access token into the Authorization header.
            String accessToken = authgear.getAccessToken();
            if (accessToken != null) {
                // example only, you can use your own networking library
                HttpURLConnection c = (HttpURLConnection) new URL(url).openConnection();
                c.setRequestProperty("Authorization", "Bearer " + accessToken);
                // ... continue making your request
            }
        }
        @Override
        public void onFailed(Throwable throwable) {
            // failed to refresh access token, the refresh token maybe expired or revoked
        }
    });
    ```
    {% endtab %}
    {% endtabs %}
    

- Handle the case that the session is revoked after SDK obtained the access token. In this case the app will call your application server with access token. But your application server will find that the access token is invalid, by checking the [resolve headers](../how-tos/backend-integration.md). Depending on your application flow, you may want to show your user login page again or reset the SDK `sessionState` to `NO_SESSION` locally. To clear the sessionState, you can use `clearSessionState` function.


    {% tabs %}
    {% tab title="Javascript" %}
    ```javascript
    // example only, if your application server return HTTP status code 401 for unauthorized request
    async function fetchAppServer() {
        var response = await authgear.fetch("YOUR_SERVER_URL");
        if (response.status === 401) {

            // if you want to clear the session state locally, call clearSessionState
            // `authgear.sessionState` will become `NO_SESSION` after calling
            await authgear.clearSessionState();
            throw new Error("user session invalid");
        }
        // ...
    }
    ```
    {% endtab %}
    {% tab title="iOS" %}
    ```swift
    // example only, if your application server return HTTP status code 401 for unauthorized request
    if let response = response as? HTTPURLResponse {
        if response.statusCode == 401 {

            // if you want to clear the session state locally, call clearSessionState
            // `authgear.sessionState` will become `.noSession` after calling
            authgear.clearSessionState { result in
                switch result {
                case .success():
                    // clear SDK session state locally
                    // `authgear.sessionState` becomes `.noSession`
                case let .failure(error):
                    // failed to clear session state
                }
            }
        }
    }
    ```
    {% endtab %}
    {% tab title="Android" %}
    ```java
    // example only, if your application server return HTTP status code 401 for unauthorized request
    responseCode = httpConn.getResponseCode();
    if (responseCode != HttpURLConnection.Unauthorized) {
        
        // if you want to clear the session state locally, call clearSessionState
        // ` authgear.getSessionState()` will become `NO_SESSION` after calling
        authgear.clearSessionState();
    }
    ```
    {% endtab %}
    {% endtabs %}
