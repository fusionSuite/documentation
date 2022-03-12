# Installation - CLI

## Introduction

A command line is available for the backend installation.

With it, you will be able to:

* standard usage:
    * create configuration with the database
    * install the data into the database
    * update the data into the database
* advanced usage:
    * create many databases configuration
    * switch to another database configuration


## Standard usage

### Create configuration

When the backend in configured (PHP, Webserver, database, see TODO: xxx), you need configure the backend to connect to the
database.

For that, run the cli with the command *env:create*. The command is:

```
./bin/cli env:create
```

The cli run in *interactive mode*, so it will ask you for the configuration. You can find the questions here:

| Question      | Description                          |
| ----------- | ------------------------------------ |
| *Enter the name of the environment configuration (examples: production, staging...)*       | :material-information-outline:  enter *production*  |
| *Select a type of database:  [1]  MySQL [2]  MariaDB [3]  PostgreSQL [4]  SQLite [5]  SQLServer Choice (1/2/3/4/5):* | :material-information-outline: enter the number related with your type of database you use  |
| *Enter the hostname of the database* | :material-information-outline: enter the hostname or the IP address of your database server |
| *Enter the name of the database* | :material-information-outline: enter the database name |
| *Enter the username to connect to the database* | :material-information-outline: enter the name of user have all access to the database defined above |
| *Enter the password to connect to the database* | :material-information-outline: enter the password of the user |
| *Enter the port to connect to the database, set 0 for the default port* | :material-information-outline: enter 0 if default port, otherwise the port number |
| *Define this environment as the current environment? (no/yes)* | :material-information-outline: enter yes to define this configuration by default |

!!! Success
    At the end, you will see the message: **The new environment has been successfully defined**.
    
    The configuration is finished

### Install

The installation process is simple.

!!! warning
    The configuration must be done before install

Run the command:

```
./bin/cli install
```

!!! Success
    Install finished if you have the 2 messages:

    * *The database is up to date*
    * *Fusionsuite is right installed / updated. Enjoy!*

??? example "Example: installation finished with success"
    ```
    using config file config/current/database.php
    using config parser php
    using migration paths 
    - /home/runner/work/backend/backend/db/migrations
    using seed paths 
    using environment testing
    using adapter mysql
    using database fusionsuite_testing
    ordering by creation time

    == 20200715182706 InitializeUsers: migrating
    == 20200715182706 InitializeUsers: migrated 0.3348s

    == 20200720091602 Properties: migrating
    == 20200720091602 Properties: migrated 0.0200s

    == 20200720091613 Types: migrating
    == 20200720091613 Types: migrated 0.0171s

    == 20200720091620 PropertyType: migrating
    == 20200720091620 PropertyType: migrated 0.0178s

    == 20200720091657 Propertylists: migrating
    == 20200720091657 Propertylists: migrated 0.0170s

    == 20200720120829 Items: migrating
    == 20200720120829 Items: migrated 0.0164s

    == 20200720120838 Itemstatus: migrating
    == 20200720120838 Itemstatus: migrated 0.0141s

    == 20200720133434 ItemProperty: migrating
    == 20200720133434 ItemProperty: migrated 0.0150s

    == 20200725083703 Relationshiptype: migrating
    == 20200725083703 Relationshiptype: migrated 0.0128s

    == 20200725083720 ItemItem: migrating
    == 20200725083720 ItemItem: migrated 0.0137s

    == 20200803061448 FusioninventoryItem: migrating
    == 20200803061448 FusioninventoryItem: migrated 0.0136s

    == 20200803061453 FusioninventoryProperty: migrating
    == 20200803061453 FusioninventoryProperty: migrated 0.0131s

    == 20200803062128 FusioninventoryDataModel: migrating
    == 20200803062128 FusioninventoryDataModel: migrated 0.0531s

    == 20200913080339 RulesCreation: migrating
    == 20200913080339 RulesCreation: migrated 0.0876s

    == 20210103122639 Propertygroups: migrating
    == 20210103122639 Propertygroups: migrated 0.0186s

    All Done. Took 0.7081s
    The database is up to date

    => The ActionScripts will be installed / updated
    Starting the process of import / update the database with scripts templates...
    -> actionZabbix.................................................................... OK 
    -> notificationDiscord............................................................. OK 
    -> notificationMail................................................................ OK 
    Fusionsuite is right installed / updated. Enjoy!
    ```


## Advanced usage

### Scripting the installation

It's posible to script the installation. Like this, the interactive mode will be dropped.

See the options on commands.

!!! example
    ```
    ./bin/cli env:create --help
        ______           _            _____       _ __     
    / ____/_  _______(_)___  ____ / ___/__  __(_) /____ 
    / /_  / / / / ___/ / __ \/ __ \__ \ / / / / / __/ _ \
    / __/ / /_/ (__  ) / /_/ / / / /__/ / /_/ / / /_/  __/
    /_/    \__,_/____/_/\____/_/ /_/____/\__,_/_/\__/\___/ 
                        
    FusionSuite Backend cli tool

    Current environment: [production]
    =======================================================

    Command env:create, version 1.0.0

    Create an enviromnent configuration

    Usage: env:create [OPTIONS...] [ARGUMENTS...]

    Arguments:
    (n/a)

    Options:
    [-c|--current]         [option] This will define this new environment as the current environment configuration
    [-d|--databasename]    The name of the database
    [-h|--help]            Show help
    [-H|--host]            The hostname of the database (IP address or DNS)
    [-n|--name]            The name of the environment configuration
    [-p|--password]        The password to connect to the database
    [-P|--port]            [option] The port to connect to the database
    [-t|--type]            The type of the database: MySQL, MariaDB, PostgreSQL, SQLite or SQLServer
    [-u|--username]        The username account to connect to the database
    [-v|--verbosity]       Verbosity level
    [-V|--version]         Show version

    Legend: <required> [optional] variadic...

    Usage Examples:
    env:create                                                                                   # create an environment configuration - interactive mode
    env:create --name staging --type MariaDB --host 127.0.0.1 --username root --password secret  # create a MariaDB staging environment configuration
    env:create -n production -t PostgreSQL -h 192.168.20.10 -u root -p secret -P 5433            # create a PostgreSQL production environment configuration
    ```

To be sure command pass without error and enter into interractive mode, **you must define all options** (the options defined with *[option]* are not mandatory).

The *usage examples* displayed into the *--help* command can guide you.


### Manage multiple configurations

#### Create configurations

Like saw previously, it's possible to create a configuration with the command *env:create*.

In fact, it's possible to create as many configurations you want.

!!! example "Examples of configurations"
    * testing
    * staging
    * production
    * client1
    * client2
    * ...

Each configuration will connect to different database.

#### List all configurations created

There is a command list all configuration you have defined:

```
./bin/cli env:list
```


#### Switch to another configuration

With the many configurations created, it can be cool to switch easily between them to test, configure...

For that, run the command:

```
./bin/cli env:switch
```

It will display all your configurations and ask to choose the configuration to apply to the current configuration.

!!! example
    ```
        ______           _            _____       _ __     
    / ____/_  _______(_)___  ____ / ___/__  __(_) /____ 
    / /_  / / / / ___/ / __ \/ __ \__ \ / / / / / __/ _ \
    / __/ / /_/ (__  ) / /_/ / / / /__/ / /_/ / / /_/  __/
    /_/    \__,_/____/_/\____/_/ /_/____/\__,_/_/\__/\___/ 
                        
    FusionSuite Backend cli tool

    Current environment: [production]
    =======================================================

    Select the environment to switch to:
    [testing]     testing
    [staging]     staging
    [production]  production
    [client1]     client1
    [client2]     client2
    Choice (testing/staging/production/client1/client2):
    ```

Of course, it's possible to script it with the option *-n xxx*.

Use the command `./bin/cli env:switch --help` to hav all information.

#### Delete a configuration

There is no cli for the moment for delete.

The usage is to delete the folder (the folder name is the name of the configuration) in folder `/config/`.

!!! caution
    If this configuration is defined to *current*, you must use the command *env:switch* to use another current configuration.

