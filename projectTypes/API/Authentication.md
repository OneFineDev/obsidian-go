
## Types
- Basic authentication
- Stateful token authentication
- Stateless token authentication
- API key authentication
- OAuth 2.0 / OpenID Connect

## Generating Auth tokens  - Stateful token authentication

1. The client sends a JSON request to a new `POST/v1/tokens/authentication` endpoint containing their credentials (email and password).
2. We look up the user record based on the email, and check if the password provided is the correct one for the user. If it’s not, then we send an error response.
3. If the password is correct, we use our `app.models.Tokens.New()` method to generate a token with an expiry time of 24 hours and the scope `"authentication"`.
4. We send this authentication token back to the client in a JSON response body.

## Using context.Context

- Every `http.Request` that our application processes has a [`context.Context`](https://golang.org/pkg/context/#Context) embedded in it, which we can use to store key/value pairs containing arbitrary data during the lifetime of the request. In this case we want to store a `User` struct containing the current user’s information.
    
- Any values stored in the request context have the type `any`. This means that after retrieving a value from the request context you need to assert it back to its original type before using it.
    
- It’s good practice to use your own custom type for the request context keys. This helps prevent naming collisions between your code and any third-party packages which are also using the request context to store information.


## Authenticating Requests

When we receive these requests, we’ll use a new `authenticate()` middleware method to execute the following logic:

- If the authentication token is not valid, then we will send the client a `401 Unauthorized` response and an error message to let them know that their token is malformed or invalid.
- If the authentication token is valid, we will look up the user details and add their details to the _request context_.
- If no `Authorization` header was provided at all, then we will add the details for an _anonymous user_ to the request context instead.

