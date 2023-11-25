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