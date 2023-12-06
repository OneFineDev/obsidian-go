
`migrate create -seq -ext=.sql -dir=./migrations create_users_table`

``` sql
CREATE TABLE IF NOT EXISTS users (
    id bigserial PRIMARY KEY,
    created_at timestamp(0) with time zone NOT NULL DEFAULT NOW(),
    name text NOT NULL,
    email citext UNIQUE NOT NULL,
    password_hash bytea NOT NULL,
    activated bool NOT NULL,
    version integer NOT NULL DEFAULT 1
);
```

1. The `email` column has the type [`citext`](https://www.postgresql.org/docs/13/citext.html) (case-insensitive text). This type stores text data exactly as it is inputted — without changing the case in any way — but _comparisons_ against the data are always case-insensitive… including lookups on associated indexes.
    
2. We’ve also got a `UNIQUE` constraint on the `email` column. Combined with the `citext` type, this means that no two rows in the database can have the same email value — even if they have different cases. This essentially enforces a database-level business rule that _no two users should exist with the same email address_.
    
3. The `password_hash` column has the type [`bytea`](https://www.postgresql.org/docs/13/datatype-binary.html) (binary string). In this column we’ll store a _one-way hash of the user’s password_ generated using [bcrypt](https://en.wikipedia.org/wiki/Bcrypt) — not the plaintext password itself.
    
4. The `activated` column stores a boolean value to denote whether a user account is ‘active’ or not. We will set this to `false` by default when creating a new user, and require the user to confirm their email address before we set it to `true`.
    
5. We’ve also included a `version` number column, which we will increment each time a user record is updated. This will allow us to use optimistic locking to prevent race conditions when updating user records, in the same way that we did with movies earlier in the book.