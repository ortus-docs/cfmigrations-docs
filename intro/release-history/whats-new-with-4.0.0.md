# What's New With 4.0.0

v4 brings a new configuration structure and file.
This pairs with new features in CFMigrations to allow for multiple named migration managers and new seeding capabilities.
Migrations will still run in v4 using the old configuration structure and location, but it is highly recommended you upgrade.

You can create the new `.cfmigrations.json` config file by running `migrate init`.

The new config file format mirrors `CFMigrations`:

```json
{
    "default": {
        "manager": "cfmigrations.models.QBMigrationManager",
        "migrationsDirectory": "resources/database/migrations/",
        "seedsDirectory": "resources/database/seeds/",
        "properties": {
            "defaultGrammar": "AutoDiscover@qb",
            "schema": "${DB_SCHEMA}",
            "migrationsTable": "cfmigrations",
            "connectionInfo": {
                "password": "${DB_PASSWORD}",
                "connectionString": "${DB_CONNECTIONSTRING}",
                "class": "${DB_CLASS}",
                "username": "${DB_USER}",
                "bundleName": "${DB_BUNDLENAME}",
                "bundleVersion": "${DB_BUNDLEVERSION}"
            }
        }
    }
}
```

More managers can be added as new top-level keys:

```json
{
    "default": {
        "manager": "cfmigrations.models.QBMigrationManager",
        "migrationsDirectory": "resources/database/migrations/",
        "seedsDirectory": "resources/database/seeds/",
        "properties": {
            "defaultGrammar": "AutoDiscover@qb",
            "schema": "${DB_SCHEMA}",
            "migrationsTable": "cfmigrations",
            "connectionInfo": {
                "password": "${DB_PASSWORD}",
                "connectionString": "${DB_CONNECTIONSTRING}",
                "class": "${DB_CLASS}",
                "username": "${DB_USER}",
                "bundleName": "${DB_BUNDLENAME}",
                "bundleVersion": "${DB_BUNDLEVERSION}"
            }
        }
    },
    "alternate": {
        "manager": "cfmigrations.models.QBMigrationManager",
        "migrationsDirectory": "resources/database/other-migrations/",
        "seedsDirectory": "resources/database/other-seeds/",
        "properties": {
            "defaultGrammar": "AutoDiscover@qb",
            "schema": "${DB_SCHEMA}",
            "migrationsTable": "cfmigrations2",
            "connectionInfo": {
                "password": "${DB_PASSWORD}",
                "connectionString": "${DB_CONNECTIONSTRING}",
                "class": "${DB_CLASS}",
                "username": "${DB_USER}",
                "bundleName": "${DB_BUNDLENAME}",
                "bundleVersion": "${DB_BUNDLEVERSION}"
            }
        }
    }
}
```

Each `migrate` command takes an optional `manager` string to use the specified configuration.

## Upgrading to v3.0.0?

Make sure to append `@qb` to the end of any qb-supplied grammars, like `AutoDiscover`.