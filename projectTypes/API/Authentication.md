
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

## Authenticating Requests

When we receive these requests, we’ll use a new `authenticate()` middleware method to execute the following logic:

- If the authentication token is not valid, then we will send the client a `401 Unauthorized` response and an error message to let them know that their token is malformed or invalid.
- If the authentication token is valid, we will look up the user details and add their details to the _request context_.
- If no `Authorization` header was provided at all, then we will add the details for an _anonymous user_ to the request context instead.