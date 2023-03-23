# Migration Files

A migration file is a component with two methods `up` and `down`. The function `up` should define how to apply the migration. The function `down` should define how to undo the change down in `up`. For `QBMigrationManager` migrations (which is the default), the `up` and `down` functions are passed an instance of `SchemaBuilder@qb` and `QueryBuilder@qb` as arguments. To learn more about the functionality and benefits of `SchemaBuilder`, `QueryBuilder`, and `qb`, please [read the QB documentation here](https://qb.ortusbooks.com/). In brief, `qb` offers a fluent, expressive syntax that can be compiled to many different database grammars, providing both readability and flexibility.

Here's the same example as above using qb's `SchemaBuilder`:

```cfc
component {

    function up( schema, qb ) {
    	schema.create( "users", function( t ) {
            t.increments( "id" );
            t.string( "email" );
            t.string( "password" );
        } );
    }

    function down( schema, qb ) {
        schema.drop( "users" );
    }

}
```

Migration files need to follow a specific naming convention — `YYYY_MM_DD_HHMISS_[describe_your_changes_here].cfc`. This is how `cfmigrations` knows in what order to run your migrations. Generating these files is made easier with the `migrate create` command from `commandbox-migrations`.

Using the injected `qb` instance, you can insert or update required data for your application.  If you want to create test data for your application, take a look at seeders below instead.

There is no limit to what you can do in a migration. It is recommended that you separate changes to different tables to separate migration files to keep things readable.


## Tips and tricks

### Setting Schema

It's important to set the `schema` attribute for `cfmigrations`.  Without it, `cfmigrations` can't tell the difference
between a migration table installed in the schema you want and any other schema on the same database.  You can
set the schema by calling the `setSchema( string schema )` method.

### Default values in MS SQL server

MS SQL server requires some special treatment when removing columns with default values. Even though syntax is almost the same, MS SQL creates a special default constraint like `DF_tablename_columname`. When migrating down, this constraint has to be removed before dropping the column. In other grammars no special named constraint is created.

Example:
```cfc
component {

    function up( schema, query   ) {
        schema.alter( "users", function ( table ) {
            table.addColumn( table.boolean( "hassuperpowers").default(0) );
        });
    }

    function down( schema, query  ) {
        schema.alter( "users", function( table ) {
            table.dropConstraint( "DF_users_hassuperpowers");
            table.dropColumn( "hassuperpowers" ) ;
        } );
    }

}
```


### Updating database content in a migration file

Sometimes you want to do multiple content updates or inserts in a migration. In this case you can use the QueryBuilder for the updates. When doing your second update you have to reset the Querybuilder object by using the newQuery method.

Example:

```cfc
component {

    function up( SchemaBuilder schema, QueryBuilder query ) {
	query.from('users')
	    .where( "username", "superuser")
	    .update( {"hassuperpowers" = true} )
	query.newQuery().from('users')
	    .where('username','RandomUser')
	    .update( {"hassuperpowers" = false} )
    }

    function down( SchemaBuilder schema, QueryBuilder query ) {
        ......
    }

}
```
