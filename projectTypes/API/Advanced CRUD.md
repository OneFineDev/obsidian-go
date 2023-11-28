- How to support _partial_ updates to a resource (so that the client only needs to send the data that they want to change).
    
- How to use _optimistic concurrency control_ to avoid race conditions when two clients try to update the same resource at the same time.
    
- How to use _context timeouts_ to terminate long-running database queries and prevent unnecessary resource use.

## Partial Updates

As we mentioned earlier in the book, when decoding the request body any fields in our `input` struct which _don’t_ have a corresponding JSON key/value pair will retain their [zero-value](https://golang.org/ref/spec#The_zero_value). We happen to check for these zero-values during validation and return the error messages that you see above.

In the context of partial update this causes a problem. How do we tell the difference between:

- A client _providing a key/value pair which has a zero-value value_ — like `{"title": ""}` — in which case we want to return a validation error.
- A client _not providing a key/value pair_ in their JSON at all — in which case we want to ‘skip’ updating the field but not send a validation error.

To help answer this, let’s quickly remind ourselves of what the zero-values are for different Go types.
<table>
<thead>
<tr>
<th>Go type</th>
<th>Zero-value</th>
</tr>
</thead>

<tbody>
<tr>
<td><code>int*</code>, <code>uint*</code>, <code>float*</code>, <code>complex</code></td>
<td><code>0</code></td>
</tr>

<tr>
<td><code>string</code></td>
<td><code>""</code></td>
</tr>

<tr>
<td><code>bool</code></td>
<td><code>false</code></td>
</tr>

<tr>
<td><code>func</code>, <code>array</code>, <code>slice</code>, <code>map</code>, <code>chan</code> and <strong>pointers</strong></td>
<td><code>nil</code></td>
</tr>
</tbody>
</table>

The key thing to notice here is that _pointers have the zero-value `nil`_.
So — in theory — we could change the fields in our `input` struct to be pointers. Then to see if a client has provided a particular key/value pair in the JSON, we can simply check whether the corresponding field in the `input` struct equals `nil` or not.

**Special Case**
One special-case to be aware of is when the client explicitly _supplies a field in the JSON request with the value `null`_. In this case, our handler will ignore the field and treat it like it hasn’t been supplied.

For example, the following request would result in no changes to the movie record (apart from the `version` number being incremented).

In most cases, it will probably suffice to explain this special-case behavior in client documentation for the endpoint and say something like _“JSON items with `null` values will be ignored and will remain unchanged”_.

## Optimistic Concurrency Control
there is a [race condition](https://stackoverflow.com/questions/34510/what-is-a-race-condition) if two clients try to update the same movie record at exactly the same time.
 This specific type of race condition is known as a _data race_. Data races can occur when two or more goroutines try to use a piece of shared data (in this example the movie record) at the same time, but the result of their operations is dependent on the exact order that the scheduler executes their instructions.

### Preventing the data race
[optimistic locking](https://stackoverflow.com/questions/129329/optimistic-vs-pessimistic-locking/129397#129397) based on the `version` number in our movie record.

The fix works like this:

1. Alice and Bob’s goroutines both call `app.models.Movies.Get()` to retrieve a copy of the movie record. Both of these records have the version number `N`.
2. Alice and Bob’s goroutines make their respective changes to the movie record.
3. Alice and Bob’s goroutines call `app.models.Movies.Update()` with their copies of the movie record. But the update is only executed _if the version number in the database is still `N`_. If it has changed, then we don’t execute the update and send the client an error message instead.
To make this work, we’ll need to change the SQL statement for updating a movie so that it looks like this:
``` sql
UPDATE movies 
SET title = $1, year = $2, runtime = $3, genres = $4, version = version + 1
WHERE id = $5 AND version = $6
RETURNING version
```
If no matching record can be found, this query will result in a `sql.ErrNoRows` error and we know that the version number has been changed (or the record has been deleted completely). Either way, it’s a form of _edit conflict_ and we can use this as a trigger to send the client an appropriate error response.

**Implementation**
1. Custom data model error
2.  update our database model’s `Update()` method to execute the new SQL query