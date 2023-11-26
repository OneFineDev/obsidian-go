SQL migrations, at a very high-level the concept works like this:

1. For every change that you want to make to your database schema (like creating a table, adding a column, or removing an unused index) you create a _pair of migration files_. One file is the ‘up’ migration which contains the SQL statements necessary to implement the change, and the other is a ‘down’ migration which contains the SQL statements to reverse (or _roll-back_) the change.
    
2. Each pair of migration files is numbered sequentially, usually `0001, 0002, 0003…` or with a [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time), to indicate the order in which migrations should be applied to a database.
    
3. You use some kind of tool or script to execute or rollback the SQL statements in the sequential migration files against your database. The tool keeps track of which migrations have already been applied, so that only the necessary SQL statements are actually executed.

Using migrations to manage your database schema, rather than manually executing the SQL statements yourself, has a few benefits:

- The database schema (along with its evolution and changes) is completely described by the ‘up’ and ‘down’ SQL migration files. And because these are just regular files containing some SQL statements, they can be included and tracked alongside the rest of your code in a version control system.
    
- It’s possible to replicate the current database schema precisely on another machine by running the necessary ‘up’ migrations. This is a big help when you need to manage and synchronize database schemas in different environments (development, testing, production, etc.).
    
- It’s possible to roll-back database schema changes if necessary by applying the appropriate ‘down’ migrations.

`migrate create -seq -ext=.sql -dir=./migrations create_movies_table`

- The `-seq` flag indicates that we want to use sequential numbering like `0001, 0002, …` for the migration files (instead of a Unix timestamp, which is the default).
- The `-ext` flag indicates that we want to give the migration files the extension `.sql`.
- The `-dir` flag indicates that we want to store the migration files in the `./migrations` directory (which will be created automatically if it doesn’t already exist).
- The name `create_movies_table` is a descriptive label that we give the migration files to signify their contents.
## Executing migrations

`migrate -path=./migrations -database=$GREENLIGHT_DB_DSN up`

#### Migrating to a specific version

As an alternative to looking at the `schema_migrations` table, if you want to see which migration version your database is currently on you can run the `migrate` tool’s `version` command, like so:

`migrate -path=./migrations -database=$EXAMPLE_DSN version 2`

You can also migrate up or down to a specific version by using the `goto` command:

`migrate -path=./migrations -database=$EXAMPLE_DSN goto 1`

#### Executing down migrations

You can use the `down` command to roll-back by a specific number of migrations. For example, to rollback the _most recent migration_ you would run:

`migrate -path=./migrations -database =$EXAMPLE_DSN down 1`

Personally, I generally prefer to use the `goto` command to perform roll-backs (as it’s more explicit about the target version) and reserve use of the `down` command for rolling-back _all migrations_, like so:

`migrate -path=./migrations -database=$EXAMPLE_DSN down`


