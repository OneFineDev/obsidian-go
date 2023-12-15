- Add checks so that only _activated users_ are able to access the various `/v1/movies**` endpoints.
    
- Implement a _permission-based authorization_ pattern, which provides fine-grained control over exactly _which users can access which endpoints_.

## Requiring User Activation

`requireActivatedUser()` middleware method to handle this. In this middleware, we want to extract the `User` struct from the request context and then check the `IsAnonymous()` method and `Activated` field to determine whether the request should continue or not.

Specifically:

- If the user is anonymous we should send a `401 Unauthorized` response and an error message saying `“you must be authenticated to access this resource”`.
- If the user is not anonymous (i.e. they have authenticated successfully and we know who they are), but they are _not activated_ we should send a `403 Forbidden` response and an error message saying `“your user account must be activated to access this resource”`.

## Permissions

<table>
<thead>
<tr>
<th>Method</th>
<th>URL Pattern</th>
<th>Required permission</th>
</tr>
</thead>

<tbody>
<tr>
<td>GET</td>
<td>/v1/healthcheck</td>
<td>—</td>
</tr>

<tr>
<td>GET</td>
<td>/v1/movies</td>
<td><code>movies:read</code></td>
</tr>

<tr>
<td>POST</td>
<td>/v1/movies</td>
<td><code>movies:write</code></td>
</tr>

<tr>
<td>GET</td>
<td>/v1/movies/:id</td>
<td><code>movies:read</code></td>
</tr>

<tr>
<td>PATCH</td>
<td>/v1/movies/:id</td>
<td><code>movies:write</code></td>
</tr>

<tr>
<td>DELETE</td>
<td>/v1/movies/:id</td>
<td><code>movies:write</code></td>
</tr>

<tr>
<td>POST</td>
<td>/v1/users</td>
<td>—</td>
</tr>

<tr>
<td>PUT</td>
<td>/v1/users/activated</td>
<td>—</td>
</tr>

<tr>
<td>POST</td>
<td>/v1/tokens/authentication</td>
<td>—</td>
</tr>
</tbody>
</table>

The relationship between permissions and users is a great example of a _many-to-many_ relationship. One user may have _many permissions_, and the same permission may belong to _many users_.

The classic way to manage a many-to-many relationship in a relational database like PostgreSQL is to create a _joining table_ between the two entities.

![[Pasted image 20231215072752.png]]

ust like the one-to-many relationship that we looked at earlier in the book, you may want to query this relationship from both sides in your database models. For example, in your database models you might want to create the following methods:

```
PermissionModel.GetAllForUser(user)       → Retrieve all permissions for a user
UserModel.GetAllForPermission(permission) → Retrieve all users with a specific permission
```
(see migration file)
- The `PRIMARY KEY (user_id, permission_id)` line sets a _composite primary key_ on our `users_permissions` table, where the primary key is made up of both the `users_id` and `permission_id` columns. Setting this as the primary key essentially means that the same user/permission combination can only appear once in the table and cannot be duplicated.
    
- When creating the `users_permissions` table we use the `REFERENCES user` syntax to create a foreign key constraint against the primary key of our `users` table, which ensures that any value in the `user_id` column has a corresponding entry in our `users` table. And likewise, we use the `REFERENCES permissions` syntax to ensure that the `permission_id` column has a corresponding entry in the `permissions` table.

## Permissions model
For now, the only thing we want to include in this model is a `GetAllForUser()` method to _return all permission codes for a specific user_. The idea is that we’ll be able to use this in our handlers and middleware like so:

## Checking Permissions
- We’ll make a new `requirePermission()` middleware which accepts a specific permission code like `"movies:read"` as an argument.
- In this middleware we’ll retrieve the current user from the request context, and call the `app.models.Permissions.GetAllForUser()` method (which we just made) to get a slice of their permissions.
- Then we can check to see if the slice contains the specific permission code needed. If it doesn’t, we should send the client a `403 Forbidden` response.

## Granting Permissions
create the `AddForUser()` method in

update our `registerUserHandler` so that new users are automatically granted the `movies:read` permission