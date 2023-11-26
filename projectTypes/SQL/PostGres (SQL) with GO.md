## Connection pools

The connection pool has four methods that we can use to configure its behavior. Let’s talk through them one-by-one.

[`SetMaxOpenConns()`](https://golang.org/pkg/database/sql/#DB.SetMaxOpenConns)
- set an upper `MaxOpenConns` limit on the number of ‘open’ connections (in-use + idle connections) in the pool.
- By default PostgreSQL has a hard limit of 100 open connections and, if this hard limit is hit under heavy load, it will cause our `pq` driver to return a `"sorry, too many clients already"` error.
- limit the number of open connections in our pool to comfortably below 100 — leaving enough headroom for any other applications or sessions that also need to use PostgreSQL.
- setting a limit comes with an important caveat. If the `MaxOpenConns` limit is reached, and all connections are in-use, then any further database tasks will be forced to wait until a connection becomes free and marked as idle. In the context of our API, the user’s HTTP request could ‘hang’ indefinitely while waiting for a free connection. So to mitigate this, it’s important to always set a timeout on database tasks using a `context.Context` object.

[`SetMaxIdleConns()`](https://golang.org/pkg/database/sql/#DB.SetMaxIdleConns)
- By default, the maximum number of idle connections is 2. In theory, allowing a higher number of idle connections in the pool will improve performance because it makes it less likely that a new connection needs to be established from scratch — therefore helping to save resources.
-  important to realize that keeping an idle connection alive comes at a cost. It takes up memory which can otherwise be used for your application and database, and it’s also possible that if a connection is idle for too long then it may become unusable. For example, by default MySQL will automatically close any connections which haven’t been used for 8 hours.
- As a guideline: _you only want to keep a connection idle if you’re likely to be using it again soon_.
- the `MaxIdleConns` limit should always be less than or equal to `MaxOpenConns`. Go enforces this and will automatically reduce the `MaxIdleConns` limit if necessary.

[`SetConnMaxLifetime()`](https://golang.org/pkg/database/sql/#DB.SetConnMaxLifetime)
- By default, there’s no maximum lifetime and connections will be reused forever.
- In theory, leaving `ConnMaxLifetime` unlimited (or setting a long lifetime) will help performance because it makes it less likely that new connections will need to be created from scratch. But in certain situations, it can be useful to enforce a shorter lifetime. For example
- If your SQL database enforces a maximum lifetime on connections, it makes sense to set `ConnMaxLifetime` to a slightly shorter value.
- To help facilitate swapping databases gracefully behind a load balancer.

 [`SetConnMaxIdleTime()`](https://golang.org/pkg/database/sql/#DB.SetConnMaxIdleTime)
 - sets the maximum length of time that a connection can be _idle_ for before it is marked as expired. By default there’s no limit.
 - If we set `ConnMaxIdleTime` to 1 hour, for example, any connections that have sat idle in the pool for 1 hour since last being used will be marked as expired and removed by the background cleanup operation.
 - This setting is really useful because it means that we can set a relatively high limit on the number of idle connections in the pool, but periodically free-up resources by removing any idle connections that we know aren’t really being used anymore.

### In Practice

1. As a rule of thumb, you should explicitly set a `MaxOpenConns` value. This should be comfortably below any hard limits on the number of connections imposed by your database and infrastructure, and you may also want to consider keeping it fairly low to act as a rudimentary throttle.
    
    For this project we’ll set a `MaxOpenConns` limit of 25 connections. I’ve found this to be a reasonable starting point for small-to-medium web applications and APIs, but ideally you should tweak this value for your hardware depending on the results of benchmarking and load-testing.
    
2. In general, higher `MaxOpenConns` and `MaxIdleConns` values will lead to better performance. But the returns are diminishing, and you should be aware that having a too-large idle connection pool (with connections that are not frequently re-used) can actually lead to reduced performance and unnecessary resource consumption.
    
    Because `MaxIdleConns` should always be less than or equal to `MaxOpenConns`, we’ll also limit `MaxIdleConns` to 25 connections for this project.
    
3. To mitigate the risk from point 2 above, you should generally set a `ConnMaxIdleTime` value to remove idle connections that haven’t been used for a long time. In this project we’ll set a `ConnMaxIdleTime` duration of 15 minutes.
    
4. It’s probably OK to leave `ConnMaxLifetime` as unlimited, unless your database imposes a hard limit on connection lifetime, or you need it specifically to facilitate something like gracefully swapping databases. Neither of those things apply in this project, so we’ll leave this as the default unlimited setting.