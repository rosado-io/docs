---
description: How to make authorized request to your application server after login with Authgear
---

# Using Authgear SDK to call your application server

In this section, we are going to explain how to make authorized request to your application server by using Authgear SDK. If you are using session cookies in your web application, you can skip this section as session cookies are handled by browser automatically.


# Setup Overview

1. To determine which user is calling your server, you will need to include Authorization header in every request that send to your application sever.
1. You will need to setup [Backend integration](../how-tos/backend-integration), Authgear will help you to resolve the Authorization header to determine whether the incoming HTTP request is authenticated or not.

In the below section, we will explain how to setup SDK to include Authorization header to your application requests.

# SDK setup
## iOS

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

- `sessionState` reflect the user logged in state. Right after configure, the session state only reflect the SDK local state. That means even `sessionState` is `authenticated`, the session maybe invalid if it is revoked remotely. You will only know that after calling the server, call `fetchUserInfo` as soon as it is proper to do so, e.g. when the device goes online.

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

- Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will refresh the access token only if it has expired. Include the access token into Authorization header of your application request.

    ```swift 
    authgear.refreshAccessTokenIfNeeded() { result in
        switch result {
        case .success():
            // access token is ready to use
            // if user is not logged in, access token will be empty
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

- Handle the case that the session is revoked after SDK obtained the access token. In this case the app will call your application server with access token. But your application server will find that the access token is invalid, by checking the [resolve headers](../how-tos/backend-integration). You may want to logout the user in the SDK in this scenario, you can call force logout.

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

    // example only, if your application server return HTTP status code 401 for unauthorized request
    if let response = response as? HTTPURLResponse {
        if response.statusCode == 401 {
            authgear.logout(force: true)
        }
    }
    ```

## Android


- Configure Authgear SDK

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

- `sessionState` reflect the user logged in state. Right after configure, the session state only reflect the SDK local state. That means even `sessionState` is `AUTHENTICATED`, the session maybe invalid if it is revoked remotely. You will only know that after calling the server, call `fetchUserInfo` as soon as it is proper to do so, e.g. when the device goes online.


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


- Call `refreshAccessTokenIfNeeded` every time before using the access token, the function will refresh the access token only if it has expired. Include the access token into Authorization header of your application request.

    ```java
    authgear.refreshAccessTokenIfNeeded(new OnRefreshAccessTokenIfNeededListener() {
        @Override
        public void onFinished() {
            // access token is ready to use
            // if user is not logged in, access token will be empty
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

- Handle the case that the session is revoked after SDK obtained the access token. In this case the app will call your application server with access token. But your application server will find that the access token is invalid, by checking the [resolve headers](../how-tos/backend-integration). You may want to logout the user in the SDK in this scenario, you can call force logout.

    ```java
    authgear.logout(true, new OnLogoutListener() {
        @Override
        public void onLogout() {
            // force logout successfully
            // authgear.getSessionState() becomes NO_SESSION
        }

        @Override
        public void onLogoutFailed(@NonNull Throwable throwable) {
            // failed to force logout
        }
    });
    ```


## Javascript

- Configure Authgear SDK

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

- `sessionState` reflect the user logged in state. Right after configure, the session state only reflect the SDK local state. That means even `sessionState` is `AUTHENTICATED`, the session maybe invalid if it is revoked remotely. You will only know that after calling the server, call `fetchUserInfo` as soon as it is proper to do so, e.g. when the device goes online.


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

- Use `authgear.fetch` to call your application server. The fetch function will include Authorization header in your application request, refresh access token will be handled automatically. `authgear.fetch` implement [fetch](https://fetch.spec.whatwg.org/).

    ```javascript
    authgear
        .fetch("YOUR_SERVER_URL")
        .then(response => response.json())
        .then(data => console.log(data));
    ```

- Handle the case that the session is revoked after SDK obtained the access token. In this case the app will call your application server with access token. But your application server will find that the access token is invalid, by checking the [resolve headers](../how-tos/backend-integration). You may want to logout the user in the SDK in this scenario, you can call force logout.

    ```javascript
    authgear
        .logout({
            force: true,
        })
        .then(() => {
            // force logout successfully
            // authgear.sessionState becomes NO_SESSION
        })
        .catch((e) => {
            // failed to force logout
        });

    // example only, if your application server return HTTP status code 401 for unauthorized request
    async function fetchAppServer() {
        var response = await authgear.fetch("YOUR_SERVER_URL");
        if (response.status === 401) {
            await authgear.logout({force: true});
            throw new Error("user logged out");
        }
        // ...
    }
    ```
