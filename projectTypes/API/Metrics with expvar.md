- How to use the `expvar` package to view application metrics in JSON format via a HTTP handler.
    
- What default application metrics are available, and how to create your own custom application metrics for monitoring the number of active goroutines and the database connection pool.
    
- How to use middleware to monitor request-level application metrics, including the counts of different HTTP response status codes.

## Custom metrics

first we need to register a custom variable with the `expvar` package, and then we need to set the value for the variable itself. In one line, the code looks roughly like this:

`expvar.NewString("version").Set(version)`

The first part of this — `expvar.NewString("version")` — creates a new [`expvar.String`](https://golang.org/pkg/expvar/#String) type, then _publishes it_ so it appears in the `expvar` handler’s JSON response with the name `"version"`, and then returns a pointer to it. Then we use the `Set()` method on it to assign an actual value to the pointer.

Two other things to note:

- The `expvar.String` type is safe for concurrent use. So — if you want to — it’s OK to manipulate this value at runtime from your application handlers.
- If you try to register two `expvar` variables with the same name, you’ll get a runtime panic when the duplicate variable is registered.