- Return the details of multiple resources in a single JSON response.
- Accept and apply optional filter parameters to narrow down the returned data set.
- Implement full-text search on your database fields using PostgreSQL’s inbuilt functionality.
- Accept and safely apply sort parameters to change the order of results in the data set.
- Develop a pragmatic, reusable, pattern to support pagination on large data sets, and return pagination metadata in your JSON responses.

## Multiple resources in one response

`/v1/movies?title=godfather&genres=crime,drama&page=1&page_size=5&sort=-year`

- The `genres` parameter will potentially contain multiple _comma-separated values_ — like `genres=crime,drama`. We will want to split these values apart and store them in a `[]string` slice.
- The `page` and `page_size` parameters will contain numbers, and we will want to convert these query string values into Go `int` types.
- There are some validation checks that we’ll want to apply to the query string values, like making sure that `page` and `page_size` are not negative numbers.
- We want our application to set some sensible _default values_ in case parameters like `page`, `page_size` and `sort` aren’t provided by the client

**Helper functions**

## Filtering
Create a filter struct and embed this in the data input struct

## Validation on Query Strings
Leverage existing validator, leverage sort-safe list for the handler.

## Listing and Filtering 
- Get all
- Use query string params to filter
Reductive filter:
``` shell
// List all movies.
/v1/movies

// List movies where the title is a case-insensitive exact match // for 'black panther'.
/v1/movies?title=black+panther

// List movies where the genres includes 'adventure'.
/v1/movies?genres=adventure

// List movies where the title is a case-insensitive exact match // for 'moana' AND the 
// genres include both 'animation' AND 'adventure'.
/v1/movies?title=moana&genres=animation,adventure
```
URL encoding :  %20 == +

Dynamic filtering:

The hardest part of building a dynamic filtering feature like this is the SQL query to retrieve the data — we need it to work with no filters, filters on both `title` and `genres`, or a filter on only one of them.

``` SQL
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (LOWER(title) = LOWER($1) OR $1 = '') 
AND (genres @> $2 OR $2 = '{}') 
ORDER BY id
```
This SQL query is designed so that each of the filters behaves like it is ‘optional’. For example, the condition `(LOWER(title) = LOWER($1) OR $1 = '')` will evaluate as `true` if the placeholder parameter `$1` is a case-insensitive match for the movie title _or_ the placeholder parameter equals `''`. So this filter condition will essentially be ‘skipped’ when movie title being searched for is the empty string `""`.

The `(genres @> $2 OR $2 = '{}')` condition works in the same way. The `@>` symbol is the ‘contains’ operator for PostgreSQL arrays, and this condition will return `true` if each value in the placeholder parameter `$2` appears in the database `genres` field _or_ the placeholder parameter contains an empty array.

**Note:** PostgreSQL also provides a range of other useful array operators and functions, including the `&&` ‘overlap’ operator, the `<@` ‘contained by’ operator, and the `array_length()` function. A complete list [can be found here](https://www.postgresql.org/docs/9.6/functions-array.html).

## Full-text Search
PostgreSQL full-text search is a powerful and highly-configurable tool, and explaining how it works and the available options in full could easily take up a whole book in itself. So we’ll keep the explanations in this chapter high-level, and focus on the practical implementation.

To implement a basic full-text search on our `title` field, we’re going to update our SQL query to look like this:
``` SQL
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (to_tsvector('simple', title) @@ plainto_tsquery('simple', $1) OR $1 = '') 
AND (genres @> $2 OR $2 = '{}')     
ORDER BY id
```

The `to_tsvector('simple', title)` function takes a movie title and splits it into _lexemes_. We specify the `simple` configuration, which means that the lexemes are just lowercase versions of the words in the title†. For example, the movie title `"The Breakfast Club"` would be split into the lexemes `'breakfast' 'club' 'the'`.

The `plainto_tsquery('simple', $1)` function takes a search value and turns it into a formatted _query term_ that PostgreSQL full-text search can understand. It normalizes the search value (again using the `simple` configuration), strips any special characters, and inserts the _and operator_ `&` between the words. As an example, the search value `"The Club"` would result in the query term `'the' & 'club'`.

The `@@` operator is the matches operator. In our statement we are using it to check whether the generated _query term matches the lexemes_. To continue the example, the query term `'the' & 'club'` will match rows which contain _both_ lexemes `'the'` and `'club'`.

### Adding indexes
see docs + code

## Sorting
Aim:
```
// Sort the movies on the title field in ascending alphabetical order.
/v1/movies?sort=title

// Sort the movies on the year field in descending numerical order.
/v1/movies?sort=-year
```
Behind the scenes we will want to translate this into an [`ORDER BY`](https://www.postgresql.org/docs/current/queries-order.html) clause in our SQL query, so that a query string parameter like `sort=-year` would result in a SQL query like this:
```SQL
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (STRPOS(LOWER(title), LOWER($1)) > 0 OR $1 = '') 
AND (genres @> $2 OR $2 = '{}')     
ORDER BY year DESC --<-- Order the result by descending year
```

The difficulty here is that the values for the `ORDER BY` clause will need to be generated at runtime based on the query string values from the client. Ideally we’d use placeholder parameters to insert these dynamic values into our query, but unfortunately it’s _not possible to use placeholder parameters for column names or SQL keywords_ (including `ASC` and `DESC`).

So instead, we’ll need to interpolate these dynamic values into our query using `fmt.Sprintf()` — making sure that the values are checked against a strict safelist first to prevent a SQL injection attack.

 We need to make sure that the order of movies is perfectly consistent between requests to prevent items in the list ‘jumping’ between the pages.

Fortunately, guaranteeing the order is simple — we just need to ensure that the `ORDER BY` clause always includes a primary key column (or another column with a unique constraint on it). So, in our case, we can apply a secondary sort on the `id` column to ensure an always-consistent order. Like so:
```SQL
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (STRPOS(LOWER(title), LOWER($1)) > 0 OR $1 = '') 
AND (genres @> $2 OR $2 = '{}')     
ORDER BY year DESC, id ASC
```

### Implementing sorting
To get the dynamic sorting working, let’s begin by updating our `Filters` struct to include some `sortColumn()` and `sortDirection()` helpers that transform a query string value (like `-year`) into values we can use in our SQL query.
## Paginating Lists
Target:
```
// Return the 5 records on page 1 (records 1-5 in the dataset)
/v1/movies?page=1&page_size=5

// Return the next 5 records on page 2 (records 6-10 in the dataset)
/v1/movies?page=2&page_size=5

// Return the next 5 records on page 3 (records 11-15 in the dataset)
/v1/movies?page=3&page_size=5
```

### The LIMIT and OFFSET clauses

The `LIMIT` clause allows you to set the maximum number of records that a SQL query should return, and `OFFSET` allows you to ‘skip’ a specific number of rows before starting to return records from the query.

if a client makes the following request:

`/v1/movies?page_size=5&page=3`

We would need to ‘translate’ this into the following SQL query:
```SQL 
SELECT id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (to_tsvector('simple', title) @@ plainto_tsquery('simple', $1) OR $1 = '') 
AND (genres @> $2 OR $2 = '{}')     
ORDER BY %s %s, id ASC
LIMIT 5 OFFSET 10
```

## Returning Pagination Metadata
### Calculating the total records

The challenging part of doing this is generating the `total_records` figure. We want this to reflect the total number of available records _given the `title` and `genres` filters that are applied_ — not the absolute total of records in the `movies` table.

A neat way to do this is to adapt our existing SQL query to include a [window function](https://www.postgresql.org/docs/current/tutorial-window.html) which counts the total number of filtered rows, like so:

```SQL
SELECT count(*) OVER(), id, created_at, title, year, runtime, genres, version
FROM movies
WHERE (to_tsvector('simple', title) @@ plainto_tsquery('simple', $1) OR $1 = '') 
AND (genres @> $2 OR $2 = '{}')     
ORDER BY %s %s, id ASC
LIMIT $3 OFFSET $4
```

The inclusion of the `count(*) OVER()` expression at the start of the query will result in the filtered record count being included as the first value in each row.
1. The `WHERE` clause is used to filter the data in the `movies` table and get the _qualifying rows_.
2. The window function `count(*) OVER()` is applied, which counts all the qualifying rows.
3. The `ORDER BY` rules are applied and the qualifying rows are sorted.
4. The `LIMIT` and `OFFSET` rules are applied and the appropriate sub-set of sorted qualifying rows is returned.