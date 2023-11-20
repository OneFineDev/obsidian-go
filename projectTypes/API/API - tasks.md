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