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
