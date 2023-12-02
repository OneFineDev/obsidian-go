Essentially, we want this middleware to check how many requests have been received in the last ‘N’ seconds and — if there have been too many — then it should send the client a `429 Too Many Requests` response. We’ll position this middleware before our main application handlers, so that it carries out this check _before_ we do any expensive processing like decoding a JSON request body or querying our database.

- About the principles behind _token-bucket_ rate-limiter algorithms and how we can apply them in the context of an API or web application.
    
- How to create middleware to rate-limit requests to your API endpoints, first by making a single rate global limiter, then extending it to support per-client limiting based on IP address.
    
- How to make rate limiter behavior configurable at runtime, including disabling the rate limiter altogether for testing purposes.

## Token bucket
- We will have a bucket that starts with `b` tokens in it.
- Each time we receive a HTTP request, we will remove one token from the bucket.
- Every `1/r` seconds, a token is added back to the bucket — up to a maximum of `b` total tokens.
- If we receive a HTTP request and the bucket is empty, then we should return a `429 Too Many Requests` response.


One of the nice things about the middleware pattern that we are using is that it is straightforward to include ‘initialization’ code which only runs once when we _wrap_ something with the middleware, rather than running on every request that the middleware handles:
```go
func (app *application) exampleMiddleware(next http.Handler) http.Handler {
    
    // Any code here will run only once, when we wrap something with the middleware. 
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        
        // Any code here will run for every request that the middleware handles.
        next.ServeHTTP(w, r)
    })
}
```

we want to add the `rateLimit()` middleware to our middleware chain. This should come after our panic recovery middleware (so that any panics in `rateLimit()` are recovered), but otherwise we want it to be used as early as possible to prevent unnecessary work for our server.

## IP-base limiting
#### Distributed applications

Using this pattern for rate-limiting will only work if your API application is running on a single-machine. If your infrastructure is distributed, with your application running on multiple servers behind a load balancer, then you’ll need to use an alternative approach.

If you’re using HAProxy or Nginx as a load balancer or reverse proxy, both of these have built-in functionality for rate limiting that it would probably be sensible to use. Alternatively, you could use a fast database like Redis to maintain a request count for clients, running on a server which all your application servers can communicate with.

## Rate limiter config
Env vars to configure Enabled, Max Reqs and Burst

