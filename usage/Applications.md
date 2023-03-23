### Installation and Uninstallation

In order to track which migrations have been ran, `cfmigrations` needs to install a tracking mechanism.  For database migrations this creates a table in your database called `cfmigrations` by default. You can do this by calling the `install()` method or by running the `migrate install` command from `commandbox-migrations`.

If you find a need to, you can uninstall the migrations tracker by calling the `uninstall()` method or by running `migrate uninstall` from `commandbox-migrations`. Running this method will rollback all ran migrations before removing the migrations tracker.

### Configuration

The module is configured by default with a single migration service that interact with your database, optionally using qb. Multiple migration services with different managers may also be configured.  The default manager for the cfmigrations is `QBMigrationManager`, but you may use others, such as those included with the `cbmongodb` and `cbelasticsearch` modules or roll your own.

The default configuration for the module settings are:

```cfc
moduleSettings = {
    "cfmigrations" : {
        "managers" : {
            "default" : {
                // The manager handling and executing the migration files
                "manager" : "cfmigrations.models.QBMigrationManager",
                // The directory containing the migration files
                "migrationsDirectory" : "/resources/database/migrations",
                // The directory containing any seeds, if applicable
                "seedsDirectory" : "/resources/database/seeds",
                // A comma-delimited list of environments which are allowed to run seeds
                "seedEnvironments" : "development",
                "properties" : {
                    "defaultGrammar" : "BaseGrammar@qb"
                }
            }
        }
    }
}
```

With this configuration, the `default` migration manager may be retrieved via WireBox at `migrationService:default`.

Here is an example of a multi-manager migrations system.  Each separate manager will require their own configuration properties.

```cfc
moduleSettings = {
    "cfmigrations": {
        "managers": {
            "db1": {
                "manager": "cfmigrations.models.QBMigrationManager",
                "migrationsDirectory": "/resources/database/db1/migrations",
                "seedsDirectory": "/resources/database/db1/seeds",
                "properties": {
                    "defaultGrammar": "MySQLGrammar@qb",
                    "datasource": "db1",
                    "useTransactions": "false",    
                }
            },
            "db2": {
                "manager": "cfmigrations.models.QBMigrationManager",
                "migrationsDirectory": "/resources/database/db2/migrations",
                "seedsDirectory": "/resources/database/db2/seeds",
                "properties": {
                    "defaultGrammar": "PostgresGrammar@qb",
                    "datasource": "db2",
                    "useTransactions": "true"    
                }
            },
            "elasticsearch": {
                "manager": "cbelasticearch.models.migrations.Manager",
                "migrationsDirectory": "/resources/elasticsearch/migrations"
            }
        }
    }
};
```

With this configuration the individual migration managers would be retreived as such:

- `db1` - getInstance( "migrationService:db1" )
- `db2` - getInstance( "migrationService:db2" )
- `elasticsearch` - getInstance( "migrationService:elasticsearch" )

### Migration Files

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

### Running Migrations

There are a few methods for working with migrations. (Each of these methods has a related command in `commandbox-migrations`.)

These methods can be run by injecting `MigrationService@cfmigrations` - for example: `getInstance( "MigrationService@cfmigrations" ).runAllMigrations( "up" )` will run all migrations.

#### `runNextMigration`

Run the next available migration in the desired direction.

| Name      | Type     | Required | Default         | Description                                                                       |
| --------- | -------- | -------- | --------------- | --------------------------------------------------------------------------------- |
| direction | String   | `true`   |                 | The direction in which to look for the next available migration — `up` or `down`. |
| postProcessHook  | function | `false`  | `function() {}` | A callback to run *after* running the migration.                                    |
| preProcessHook   | function | `false`  | `function() {}` | A callback to run *before* running the migration.                                    |

#### `runAllMigrations`

Run all available migrations in the desired direction.

| Name      | Type     | Required | Default         | Description                                                               |
| --------- | -------- | -------- | --------------- | ------------------------------------------------------------------------- |
| direction | String   | `true`   |                 | The direction for which to run the available migrations — `up` or `down`. |
| postProcessHook  | function | `false`  | `function() {}` | A callback to run *after* running each migration.                                    |
| preProcessHook   | function | `false`  | `function() {}` | A callback to run *before* running each migration.                                    |

#### `reset`

Returns the database to an empty state by dropping all objects.

#### `findAll`

Returns an array of **all** migrations:

```js
[{
	fileName = "2019_12_18_195831_create-users-table.cfc",
	componentName = "2019_12_18_195831_create-users-table",
	absolutePath = "/var/www/html/app/resources/migrations/2019_12_18_195831_create-users-table.cfc",
	componentPath = "/app/resources/migrations/2019_12_18_195831_create-users-table.cfc",
	timestamp = 123455555,
	migrated = false,
	canMigrateUp = true,
	canMigrateDown = false,
	migratedDate = "2019-03-22"
}]
```

#### `hasMigrationsToRun`

Returns `true` if there are available migrations which can be run in the provided order.

| Name      | Type     | Required | Default         | Description                                                               |
| --------- | -------- | -------- | --------------- | ------------------------------------------------------------------------- |
| direction | String   | `true`   |                 | The direction for which to run the available migrations — `up` or `down`. |

### Seeders

Seeding your database is an optional step that allows you to add data to your database in mass.  It is usually used in development to create a populated environment for testing.  Seeders should not be used for data required to run your application or to migrate data between columns.  Seeders should be seen as entirely optional.  If a seeder is never ran, your application should still work.

Seeders can be ran by calling the `seed` method on a `MigrationService`.  It takes an optional `seedName` string to only run a specific seeder. Additionally, you can run all your seeders when migrating your database by passing `seed = true` to the `up` method.

By default, seeders can only be ran in `development` environments.  This can be configured on each `manager` by setting a `seedEnvironments` key to either a list or array of allowed environments to run in.

A seeder is a cfc file with a single required method - `run`.  For the `QBMigrationManager`, it is passed a `QueryBuilder` instance and a `MockData` instance, useful for creating fake data to insert into your database. (Other Migration Managers will have other arguments passed. Please refer to the documentation for your specific manager.)

```cfc
component {

    function run( qb, mockdata ) {
        qb.table( "users" ).insert(
            mockdata.mock(
                $num = 25,
                "firstName": "firstName",
                "lastName": "lastName",
                "email": "email",
                "password": "string-secure"
            )
        );
    }

}
```