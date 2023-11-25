# JSON Responses
## JSON encoding
<table>
<thead>
<tr>
<th>Go type</th>
<th>⇒</th>
<th>JSON type</th>
</tr>
</thead>

<tbody>
<tr>
<td><code>bool</code></td>
<td>⇒</td>
<td>JSON boolean</td>
</tr>

<tr>
<td><code>string</code></td>
<td>⇒</td>
<td>JSON string</td>
</tr>

<tr>
<td><code>int*</code>, <code>uint*</code>, <code>float*</code>, <code>rune</code></td>
<td>⇒</td>
<td>JSON number</td>
</tr>

<tr>
<td><code>array</code>, <code>slice</code></td>
<td>⇒</td>
<td>JSON array</td>
</tr>

<tr>
<td><code>struct</code>, <code>map</code></td>
<td>⇒</td>
<td>JSON object</td>
</tr>

<tr>
<td><code>nil</code> pointers, interface values, slices, maps, etc.</td>
<td>⇒</td>
<td>JSON null</td>
</tr>

<tr>
<td><code>chan</code>, <code>func</code>, <code>complex*</code></td>
<td>⇒</td>
<td>Not supported</td>
</tr>

<tr>
<td><code>time.Time</code></td>
<td>⇒</td>
<td>RFC3339-format JSON string</td>
</tr>

<tr>
<td><code>[]byte</code></td>
<td>⇒</td>
<td>Base64-encoded JSON string</td>
</tr>
</tbody>
</table>

Two methods to do this:
`json.Marshal`
`json.NewEncoder(w writer).Encode(data any)` - this is simpler but denies the opportunity to set headers as the result is immediately written to `w`.

## Hiding struct fields in the JSON object
The `-` (hyphen) directive can be used when you _never_ want a particular struct field to appear in the JSON output. This is useful for fields that contain internal system information that isn’t relevant to your users, or sensitive information that you don’t want to expose (like the hash of a password).

In contrast the `omitempty` directive hides a field in the JSON output _if and only if_ the struct field value is empty, where empty is defined as being:

- Equal to `false`, `0`, or `""`
- An empty `array`, `slice` or `map`
- A `nil` pointer or a `nil` interface value
## Enveloping responses
``` go

type envelope map[string]any

err = app.writeJSON(w, http.StatusOK, envelope{"movie": m}, nil)

```

## JSON Customization - A type with a method to do custom JSO transformations

A clean and simple approach is to create a custom type specifically for the `Runtime` field, and implement a `MarshalJSON()` method on this custom type.

If your `MarshalJSON()` method returns a JSON string value, like ours does, then you must wrap the string in double quotes before returning it. Otherwise it won’t be interpreted as a _JSON string_ and you’ll receive a runtime

Deliberately using a _value receiver_ for our `MarshalJSON()` method rather than a _pointer receiver_ like `func (r *Runtime) MarshalJSON()`. This gives us more flexibility because it means that our custom JSON encoding will work on _both_ `Runtime` values and pointers to `Runtime` values.

## JSON Error Responses
Replace `http.Error()` and `http.NotFound()` with custom JSON error logger functions

## Custom router error handlers
httprouter allows us to set our own custom error handlers when we initialize the router

``` gO
router.NotFound = http.HandlerFunc(app.notFoundResponse)
```

## Panic Recovery
Can wrap router in panic recovery middleware

# JSON Requests

## JSON Decoding 
 there are two approaches that you can take to _decode_ JSON into a nativeGo object: using a [`json.Decoder`](https://golang.org/pkg/encoding/json/#Decoder) type or using the [`json.Unmarshal()`](https://golang.org/pkg/encoding/json/#Unmarshal) function.
 using `json.Decoder` is generally the best choice. It’s more efficient than `json.Unmarshal()`, requires less code, and offers some helpful settings that you can use to tweak its behavior.

## Zero Values
As you might have guessed, when we do this the `Year` field in our `input` struct is left with its zero value (which happens to be `0` because the `Year` field is an `int32` type).

This leads to an interesting question: how can you tell the difference between a client _not providing a key/value pair_, and _providing a key/value pair but deliberately setting it to its zero value_? Like this:
```go
$ BODY='{"title":"Moana","year":0,"runtime":107, "genres":["animation","adventure"]}'
$ curl -d "$BODY" localhost:4000/v1/movies
{Title:Moana Year:0 Runtime:107 Genres:[animation adventure]}
```

## JSON decoding type transformation
<table>
<thead>
<tr>
<th>JSON type</th>
<th>⇒</th>
<th>Supported Go types</th>
</tr>
</thead>

<tbody>
<tr>
<td>JSON boolean</td>
<td>⇒</td>
<td><code>bool</code></td>
</tr>

<tr>
<td>JSON string</td>
<td>⇒</td>
<td><code>string</code></td>
</tr>

<tr>
<td>JSON number</td>
<td>⇒</td>
<td><code>int*</code>, <code>uint*</code>, <code>float*</code>, <code>rune</code></td>
</tr>

<tr>
<td>JSON array</td>
<td>⇒</td>
<td><code>array</code>, <code>slice</code></td>
</tr>

<tr>
<td>JSON object</td>
<td>⇒</td>
<td><code>struct</code>, <code>map</code></td>
</tr>
</tbody>
</table>

## JSON Unmarshal
Basically more verbose and less performant

## Managing bad requests
Triaging the `Decode()` error

<table>
<thead>
<tr>
<th>Error types</th>
<th>Reason</th>
</tr>
</thead>

<tbody>
<tr>
<td><a href="https://golang.org/pkg/encoding/json/#SyntaxError"><code>json.SyntaxError</code></a> <br><a href="https://golang.org/pkg/io/#pkg-variables"><code>io.ErrUnexpectedEOF</code></a></td>
<td>There is a syntax problem with the JSON being decoded.</td>
</tr>

<tr>
<td><a href="https://golang.org/pkg/encoding/json/#UnmarshalTypeError"><code>json.UnmarshalTypeError</code></a></td>
<td>A JSON value is not appropriate for the destination Go type.</td>
</tr>

<tr>
<td><a href="https://golang.org/pkg/encoding/json/#InvalidUnmarshalError"><code>json.InvalidUnmarshalError</code></a></td>
<td>The decode destination is not valid (usually because it is not a pointer). This is actually a problem with our application code, not the JSON itself.</td>
</tr>

<tr>
<td><a href="https://golang.org/pkg/io/#pkg-variables"><code>io.EOF</code></a></td>
<td>The JSON being decoded is empty.</td>
</tr>
</tbody>
</table>

Triaging these potential errors (which we can do using Go’s [`errors.Is()`](https://golang.org/pkg/errors/#Is) and [`errors.As()`](https://golang.org/pkg/errors/#As) functions) is going to make the code in our `createMovieHandler` _a lot_ longer and more complicated. And the logic is something that we’ll need to duplicate in other handlers throughout this project too.

So, to assist with this, let’s create a new `readJSON()` helper in the `cmd/api/helpers.go` file. In this helper we’ll decode the JSON from the request body as normal, then triage the errors and replace them with our own custom messages as necessary.