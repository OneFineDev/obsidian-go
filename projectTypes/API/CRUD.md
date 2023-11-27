## Models, or Data Access Layer
If you don’t like the term _model_ then you might want to think of this as your _data access_ or _storage_ layer instead. But whatever you prefer to call it, the principle is the same — it will encapsulate all the code for reading and writing movie data to and from our PostgreSQL database.

Prep:
1. Let’s head back to the `internal/data/movies.go` file and create a `MovieModel` struct type and some placeholder methods for performing basic CRUD (create, read, update and delete) actions against our `movies` database table.
2. As an additional step, we’re going to wrap our `MovieModel` in a parent `Models` struct. Doing this is totally optional, but it has the benefit of giving you a convenient single ‘container’ which can hold and represent _all_ your database models as your application grows.
3. And now, let’s edit our `cmd/api/main.go` file so that the `Models` struct is initialized in our `main()` function, and then passed to our handlers as a dependency.


One of the nice things about this pattern is that the code to execute actions on our `movies` table will be very clear and readable from the perspective of our API handlers. For example, we’ll be able to execute the `Insert()` method by simply writing:
`app.models.Movies.Insert(…)`

# Go’s [`database/sql`](https://golang.org/pkg/database/sql/) package (as opposed to GORM)
## Insert

Postgres: At the end of the query we have a [`RETURNING`](https://www.postgresql.org/docs/current/dml-returning.html) clause. This is a PostgreSQL-specific clause (it’s not part of the SQL standard) that you can use to return values from any record that is being manipulated by an `INSERT`, `UPDATE` or `DELETE` statement.

Normally, you would use Go’s [`Exec()`](https://golang.org/pkg/database/sql/#DB.Exec) method to execute an `INSERT` statement against a database table. But because our SQL query is returning a single row of data (thanks to the `RETURNING` clause), we’ll need to use the [`QueryRow()`](https://golang.org/pkg/database/sql/#DB.QueryRow) method here instead.
:::
Because the `Insert()` method signature takes a `*Movie` pointer as the parameter, when we call `Scan()` to read in the system-generated data we’re updating the values _at the location the parameter points to_. Essentially, our `Insert()` method _mutates_ the `Movie` struct that we pass to it and adds the system-generated values to it.

Note the `pq.Array()` method for handling array input into postgres.

Then hook the `movie.Insert()` method to the corresponding handler. 