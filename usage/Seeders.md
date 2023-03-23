# Seeders

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