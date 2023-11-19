## Basic Project Structure
```
├── bin
├── cmd
│ └── api
│     └── main.go
├── internal
├── migrations
├── remote
├── go.mod
└── Makefile
```

## Define Endpoints

<table>
<thead>
<tr>
<th>Method</th>
<th>URL Pattern</th>
<th>Action</th>
</tr>
</thead>

<tbody>
<tr>
<td>GET</td>
<td>/v1/healthcheck</td>
<td>Show application health and version information</td>
</tr>

<tr>
<td>GET</td>
<td>/v1/movies</td>
<td>Show the details of all movies</td>
</tr>

<tr>
<td>POST</td>
<td>/v1/movies</td>
<td>Create a new movie</td>
</tr>

<tr>
<td>GET</td>
<td>/v1/movies/:id</td>
<td>Show the details of a specific movie</td>
</tr>

<tr>
<td>PATCH</td>
<td>/v1/movies/:id</td>
<td>Update the details of a specific movie</td>
</tr>

<tr>
<td>DELETE</td>
<td>/v1/movies/:id</td>
<td>Delete a specific movie</td>
</tr>

<tr>
<td>POST</td>
<td>/v1/users</td>
<td>Register a new user</td>
</tr>

<tr>
<td>PUT</td>
<td>/v1/users/activated</td>
<td>Activate a specific user</td>
</tr>

<tr>
<td>PUT</td>
<td>/v1/users/password</td>
<td>Update the password for a specific user</td>
</tr>

<tr>
<td>POST</td>
<td>/v1/tokens/authentication</td>
<td>Generate a new authentication token</td>
</tr>

<tr>
<td>POST</td>
<td>/v1/tokens/password-reset</td>
<td>Generate a new password-reset token</td>
</tr>

<tr>
<td>GET</td>
<td>/debug/vars</td>
<td>Display application metrics</td>
</tr>
</tbody>
</table>

## func main()

``` go
func main() {
    var cfg config
    
    flag.IntVar(&cfg.port, "port", 4000, "API Server Port ")
    flag.StringVar(&cfg.env, "env", "dev", "Environment (dev|sit|pre|prd)")
    flag.Parse()
    
    logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
        AddSource: true,
        Level:     slog.LevelDebug,
    }))
    
    app := &application{
        config: cfg,
        logger: logger,
    }
    
    mux := http.NewServeMux()
    mux.HandleFunc("/v1/healthcheck", app.healthcheckHanddler)
    
    srv := &http.Server{
        Addr:         fmt.Sprintf(":%d", cfg.port),
        Handler:      mux,
        IdleTimeout:  time.Minute,
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
        ErrorLog:     slog.NewLogLogger(logger.Handler(), slog.LevelError),
    }
    
    logger.Info("starting server", "addr", srv.Addr, "env", cfg.env)
    
    err := srv.ListenAndServe()
    logger.Error(err.Error())
    os.Exit(1)
}
```

## Making the healthcheck handler a method on the application struct

`healthcheckHandler` is implemented as a method on our application struct.
This is an effective and idiomatic way to make dependencies available to our handlers without resorting to global variables or closures — any dependency that the `healthcheckHandler` needs can simply be included as a field in the application struct when we initialize it in `main()`.
We can see this pattern already being used in the code above, where the operating environment name is retrieved from the application struct by calling `app.config.env`.

