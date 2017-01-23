Data Migrate
====

Run data migrations alongside schema migrations.

What's it do?
-------------

Data migrations are stored in db/data. They act like schema
migrations, except they should be reserved for data migrations. For
instance, if you realize you need to titleize all yours titles, this
is the place to do it. Running any of the provided rake tasks also
creates a data schema table to mirror the usual schema migrations
table to track all the goodness.

Data migrations can be created at the same time as schema migrations,
or independently.  Database (db:) tasks have been added and extended
to run on data migrations only, or in conjunction with the schema
migration.  For instance, `rake db:migrate:with_data` will run both
schema and data migrations in the proper order.

Note: If a data and schema migration share the same version number, schema gets precedence when migrating up. Data does down.

Rails 3 and Ruby 1.9
--------------------

Data Migrate is Rails 3 and Ruby 1.9 compatible.

I made a readme change!

Installation
------------
Add the gem to your project

    # Gemfile
    gem 'data_migrate'

Then `bundle install` and you are ready to go.

So you know, when you use one of the provide rake tasks, a table
called 'data_migrations' will be created in your database. This
is to mirror the way the standard 'db' rake tasks work. If you've
installed previous to v1.1.0, you'll want to delete the
'create\_data\_migrations_table' migration.

Usage
-----

### Generating Migrations

You can generate a data migration as you would a schema migration:

    rails g data_migration add_this_to_that

By default, the migration also generates a schema migration by the same name.
This allows you to do things like:

    rails g data_migration add_this_to_that this:string

If you need a data only migration, either run it as such, with the skip-schema-migration flag:

    rails g data_migration add_this_to_that --skip-schema-migration

A shorter alias for this is -m:

    rails g data_migration add_this_to_that -m

To ensure backwards compatibility, you should inform data-migrate which classes will be used
in the migration. data-migrate will create a secondary file recording the current
state of those model classes and will ensure that whenever the migration is run,
the classes will be correct.

    rails g data_migration add_this_to_that --classes=User Credential Email

### Required Migrations

You can specify certain migrations as being "required" - the idea is that these
need to run before the app can come up, while normal migrations can run
once the deploy is finished. These will show up
in db/required_data instead of in db/data, though the version will still be
stored in the same data_migrations table. Running rake data:migrate normally
will only run *non-required* migrations (this seems backwards but this is done
because required migrations are rare.

There is no real difference between required and non/required - it's just an
easy way to differentiate two different types of data migrations.

### Rake Tasks

    $> rake -T data
    rake data:forward                 # Pushes the schema to the next version (specify steps w/ STEP=n)
    rake data:migrate                 # Migrate non-required data migrations (options: VERSION=x, VERBOSE=false)
    rake data:migrate:down            # Runs the "down" for a given migration VERSION
    rake data:migrate:redo            # Rollbacks the database one migration and re migrate up (options: STEP=x, VERSIO...
    rake data:migrate:status          # Display status of data migrations
    rake data:migrate:up              # Runs the "up" for a given migration VERSION
    rake data:migrate:required        # Run "required" migrations
    rake data:migrate:all             # Run both required and normal migrations
    rake data:rollback                # Rolls the schema back to the previous version (specify steps w/ STEP=n)
    rake data:version                 # Retrieves the current schema version number for data migrations
    rake db:forward:with_data         # Pushes the schema to the next version (specify steps w/ STEP=n)
    rake db:migrate:down:with_data    # Runs the "down" for a given migration VERSION
    rake db:migrate:redo:with_data    # Rollbacks the database one migration and re migrate up (options: STEP=x, VERSIO...
    rake db:migrate:status:with_data  # Display status of data and schema migrations
    rake db:migrate:up:with_data      # Runs the "up" for a given migration VERSION
    rake db:migrate:with_data         # Migrate the database data and schema (options: VERSION=x, VERBOSE=false)
    rake db:rollback:with_data        # Rolls the schema back to the previous version (specify steps w/ STEP=n)
    rake db:version:with_data         # Retrieves the current schema version numbers for data and schema migrations

Tasks work as they would with the 'vanilla' db version.  The 'with_data' addition to the 'db' tasks will run the task in the context of both the data and schema migrations.  That is, `rake db:rollback:with_data` will check to see if it was a schema or data migration invoked last, and do that.  Tasks invoked in that space also have an additional line of output, indicating if the action is performed on data or schema.

With 'up' and 'down', you can specify the option 'BOTH', which defaults to false. Using true, will migrate both the data and schema (in the desired direction) if they both match the version provided.  Again, going up, schema is given precedence. Down its data.

For more example, assume you have the 2 files:
  db/migrate/20110419021211_add_x_to_y.rb
  db/data/20110419021211_add_x_to_y.rb

Running `rake db:migrate:up:with_data VERSION=20110419021211` would execute the 'db/migrate' version.
Running `rake db:migrate:up:with_data VERSION=20110419021211` would execute the 'db/migrate' version, followed by the 'db/data' version.

Going down instead of up would be the opposite.

`rake db:migrate:status:with_data` provides and additional column to indicate which type of migration.

Thanks
------
[Jeremy Durham](http://jeremydurham.com/) for fleshing out the idea with me, and providing guidance.
You!  Yes, you. Thanks for checking it out.
