`go get golang.org/x/crypto/bcrypt@latest`

- password hashing and checking
- input validation
- user model

## Activation

To give you an overview upfront, the account activation process will work like this:

1. As part of the registration process for a new user we will create a cryptographically-secure random _activation token_ that is impossible to guess.
2. We will then store a hash of this activation token in a new `tokens` table, alongside the new user’s ID and an expiry time for the token.
3. We will send the original (unhashed) activation token to the user in their welcome email.
4. The user subsequently submits their token to a new `PUT /v1/users/activated` endpoint.
5. If the hash of the token exists in the `tokens` table and hasn’t expired, then we’ll update the `activated` status for the relevant user to `true`.
6. Lastly, we’ll delete the activation token from our `tokens` table so that it can’t be used again.

## table

```sql
CREATE TABLE IF NOT EXISTS tokens (
    hash bytea PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users ON DELETE CASCADE,
    expiry timestamp(0) with time zone NOT NULL,
    scope text NOT NULL
);
```

- The `hash` column will contain a SHA-256 hash of the activation token. It’s important to emphasize that we will only store a hash of the activation token in our database — not the activation token itself.
    
    We want to hash the token before storing it for the same reason that we bcrypt a user’s password — it provides an extra layer of protection if the database is ever compromised or leaked. Because our activation token is going to be a high-entropy random string (128 bits) — rather than something low entropy like a typical user password — [it is sufficient](https://security.stackexchange.com/questions/151257/what-kind-of-hashing-to-use-for-storing-rest-api-tokens-in-the-database) to use a fast algorithm like SHA-256 to create the hash, instead of a slow algorithm like bcrypt.
    
- The `user_id` column will contain the ID of the user associated with the token. We use the `REFERENCES user` syntax to create a [foreign key constraint](https://www.postgresqltutorial.com/postgresql-foreign-key/) against the primary key of our `users` table, which ensures that any value in the `user_id` column has a corresponding `id` entry in our `users` table.
    
    We also use the `ON DELETE CASCADE` syntax to instruct PostgreSQL to _automatically delete all records for a user in our `tokens` table when the parent record in the `users` table is deleted_.
- The `expiry` column will contain the time that we consider a token to be ‘expired’ and no longer valid. Setting a short expiry time is good from a security point-of-view because it helps reduce the window of possibility for a successful brute-force attack against the token. And it also helps in the scenario where the user is _sent a token but doesn’t use it_, and their email account is compromised at a later time. By setting a short time limit, it reduces the time window that the compromised token could be used.
    
    Of course, the security risks here need to be weighed up against usability, and we want the expiry time to be long enough for a user to be able to activate the account at their leisure. In our case, we’ll set the expiry time for our activation tokens to 3 days from the moment the token was created.
    
- Lastly, the `scope` column will denote what _purpose_ the token can be used for. Later in the book we’ll also need to create and store _authentication tokens_, and most of the code and storage requirements for these is exactly the same as for our activation tokens. So instead of creating separate tables (and the code to interact with them), we’ll store them in one table with a value in the `scope` column to restrict the purpose that the token can be used for.
## Activation

1. The user submits the plaintext activation token (which they just received in their email) to the `PUT /v1/users/activated` endpoint.
2. We validate the plaintext token to check that it matches the expected format, sending the client an error message if necessary.
3. We then call the `UserModel.GetForToken()` method to retrieve the details of the user associated with the provided token. If there is no matching token found, or it has expired, we send the client an error message.
4. We activate the associated user by setting `activated = true` on the user record and update it in our database.
5. We delete all activation tokens for the user from the `tokens` table. We can do this using the `TokenModel.DeleteAllForUser()` method that we made earlier.
6. We send the updated user details in a JSON response.