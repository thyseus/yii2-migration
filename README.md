# yii2-migration

![Latest Stable Version](https://img.shields.io/packagist/v/bizley/migration.svg)
[![Total Downloads](https://img.shields.io/packagist/dt/bizley/migration.svg)](https://packagist.org/packages/bizley/migration)
![License](https://img.shields.io/packagist/l/bizley/migration.svg)

## Migration creator and updater

Generates migration file based on the existing database table and previous migrations.

## Installation

Add the package to your composer.json (this version is in beta stage so use this for now):

    {
        "require": {
            "bizley/migration": "*"
        }
    }

and run `composer update` or alternatively run `composer require bizley/migration`

## Basic configuration

Add the following in your configuration file (preferably console configuration file):

    'components' => [
        // ...
    ],
    'controllerMap' => [
        'migration' => [
            'class' => 'bizley\migration\controllers\MigrationController',
        ],
    ],

## Usage

Run console command

    php yii migration/create table_name

to generate migration to create DB table `table_name`.

Run console command

    php yii migration/update table_name

to generate migration to update DB table `table_name`.

You can generate multiple migrations for many tables at once by separating the names with a comma:

    php yii migration table_name1,table_name2,table_name3

In case the file to be generated already exists user is asked if it should be overwritten or appended with number.

## Updating migration

With yii2-migration 2.0 it is possible to generate update migration for database table.

1. History of applied migrations is scanned to gather all modifications made to the table.
2. Virtual table schema is prepared and compared with current table schema.
3. Differences are generated as update migration.
4. In case of migration history not keeping information about the table create migration is generated.

## Command line parameters

    --migrationPath -p

Directory storing the migration classes. _(default '@app/migrations')_

    --migrationNamespace -n

Namespace in case of generating namespaced migration. _(default null)_  
With this option set `migrationPath` is ignored. 

    --defaultDecision -d

Default decision what to do in case the file to be generated already exists. _(default 'n')_  
Available options are:

- 'y' = asks before every existing file, overwrite is default option,
- 'n' = asks before every existing file, skip is default option,
- 'a' = asks before every existing file, append next number is default option,
- 'o' = doesn't ask, all existing files are overwritten,
- 's' = doesn't ask, no existing files are overwritten,
- 'p' = doesn't ask, all existing files are appended with next number.

Both `create` and `update` action use the same decision mechanism.
 
    --templateFile -F

Template file for generating create migrations. _(default '@vendor/bizley/migration/src/views/create_migration.php')_

    --templateFileUpdate -U

Template file for generating update migrations. _(default '@vendor/bizley/migration/src/views/update_migration.php')_

    --useTablePrefix -P

Whether the table names generated should consider the `tablePrefix` setting of the DB connection. _(default 1)_

    --db

Application component's ID of the DB connection to use when generating migrations. _(default 'db')_

    --migrationTable -t

Name of the table for keeping applied migration information. _(default '{{%migration}}')_  
The same as in yii\console\controllers\MigrateController::$migrationTable.

    --migrationNamespaces -N

List of namespaces containing the migration classes. _(default [])_  
The same as in yii\console\controllers\BaseMigrateController::$migrationNamespaces.

    --showOnly -s

Whether to only display changes instead of generating update migration. _(default 0)_

    --generalSchema -g

Whether to use general column schema instead of database specific. _(default 0)_

> ### MySQL examples:  
> Column `varchar(45)`  
> generalSchema=0: `$this->string(45)`    
> generalSchema=1: `$this->string()`  
> Column `int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY`    
> generalSchema=0: `$this->integer(11)->notNull()->append('AUTO_INCREMENT PRIMARY KEY')`  
> generalSchema=1: `$this->primaryKey()`  

Remember that with different database types general column schemas may be generated with different length.

    --fixHistory -h
    
Whether to add migration history entry when migration is generated. _(default 0)_


## Notes

This extension is tested on MySQL database but should work with all database types supported in Yii 2 core:

- CUBRID (9.3.x and higher)
- MS SQL Server (2008 and above)
- MySQL (4.1.x and 5.x)
- Oracle
- PostgreSQL (9.x and above)
- SQLite (2/3)

Let me know if something is wrong with databases other than MySQL (and in case of MySQL let me know as well).

As far as I know Yii 2 does not keep information about table indexes (except unique ones) and foreign keys' 
ON UPDATE and ON DELETE actions so unfortunately this can not be tracked and applied to generated migrations - 
you have to add it on your own.

Only history of migrations extending `yii\db\Migration` class can be properly scanned and only changes applied with
default `yii\db\Migration` methods can be recognised (with the exception of `execute()`, `addCommentOnTable()` and 
`dropCommentFromTable()` methods). Changes made to table's data (like `insert()`, `delete()`, `truncate()`, etc.) 
are not tracked.
