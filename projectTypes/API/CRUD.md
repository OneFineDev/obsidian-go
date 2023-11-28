## Models, or Data Access Layer
If you don’t like the term _model_ then you might want to think of this as your _data access_ or _storage_ layer instead. But whatever you prefer to call it, the principle is the same — it will encapsulate all the code for reading and writing movie data to and from our PostgreSQL database.

Prep:
1. Let’s head back to the `internal/data/movies.go` file and create a `MovieModel` struct type and some placeholder methods for performing basic CRUD (create, read, update and delete) actions against our `movies` database table.
2. As an additional step, we’re going to wrap our `MovieModel` in a parent `Models` struct. Doing this is totally optional, but it has the benefit of giving you a convenient single ‘container’ which can hold and represent _all_ your database models as your application grows.
3. And now, let’s edit our `cmd/api/main.go` file so that the `Models` struct is initialized in our `main()` function, and then passed to our handlers as a dependency.


One of the nice things about this pattern is that the code to execute actions on our `movies` table will be very clear and readable from the perspective of our API handlers. For example, we’ll be able to execute the `Insert()` method by simply writing:
`app.models.Movies.Insert(…)`

# Go’s [`database/sql`](https://golang.org/pkg/database/sql/) package (as opposed to GORM)
## Create

Postgres: At the end of the query we have a [`RETURNING`](https://www.postgresql.org/docs/current/dml-returning.html) clause. This is a PostgreSQL-specific clause (it’s not part of the SQL standard) that you can use to return values from any record that is being manipulated by an `INSERT`, `UPDATE` or `DELETE` statement.

Normally, you would use Go’s [`Exec()`](https://golang.org/pkg/database/sql/#DB.Exec) method to execute an `INSERT` statement against a database table. But because our SQL query is returning a single row of data (thanks to the `RETURNING` clause), we’ll need to use the [`QueryRow()`](https://golang.org/pkg/database/sql/#DB.QueryRow) method here instead.
:::
Because the `Insert()` method signature takes a `*Movie` pointer as the parameter, when we call `Scan()` to read in the system-generated data we’re updating the values _at the location the parameter points to_. Essentially, our `Insert()` method _mutates_ the `Movie` struct that we pass to it and adds the system-generated values to it.

Note the `pq.Array()` method for handling array input into postgres.

Then hook the `movie.Insert()` method to the corresponding handler. 

**Extra Info**
#### $N notation
A nice feature of the PostgreSQL placeholder parameter `$N` notation is that you can use the same parameter value in multiple places in your SQL statement. For example, it’s perfectly acceptable to write code like this:
``` go
// This SQL statement uses the $1 parameter twice, and the value `123` will be used in 
// both locations where $1 appears.
stmt := "UPDATE foo SET bar = $1 + $2 WHERE bar = $1"
err := db.Exec(stmt, 123, 456)
if err != nil {
    …
}
```
#### Executing multiple statements
Occasionally you might find yourself in the position where you want to execute more than one SQL statement in the same database call, like this:
```go
stmt := `
    UPDATE foo SET bar = true;    UPDATE foo SET baz = false;`

err := db.Exec(stmt)
if err != nil {
    …
}
```

Having multiple statements in the same call is supported by the `pq` driver, _so long as the statements do not contain any placeholder parameters_. If they do contain placeholder parameters, then you’ll receive the following error message at runtime.
To work around this, you will need to either split out the statements into separate database calls, or if that’s not possible, you can create a [custom function](https://www.postgresql.org/docs/current/xfunc-sql.html) in PostgreSQL which acts as a wrapper around the multiple SQL statements that you want to run.
## Read


## Update
1. Extract the movie ID from the URL using the `app.readIDParam()` helper.
2. Fetch the corresponding movie record from the database using the `Get()` method that we made in the previous chapter.
3. Read the JSON request body containing the updated movie data into an `input` struct.
4. Copy the data across from the `input` struct to the movie record.
5. Check that the updated movie record is valid using the `data.ValidateMovie()` function.
6. Call the `Update()` method to store the updated movie record in our database.
7. Write the updated movie data in a JSON response using the `app.writeJSON()` helper.
## Delete
- If a movie with the `id` provided in the URL exists in the database, we want to delete the corresponding record and return a success message to the client.
- If the movie `id` doesn’t exist, we want to return a `404 Not Found` response to the client.