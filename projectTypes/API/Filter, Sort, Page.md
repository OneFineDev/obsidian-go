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

