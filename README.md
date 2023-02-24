# Liquibase
Database Migrations with Liquibase
This project demonstrates database refactoring using Liquibase commands against postgres database. Here are two options for using Liquibase as a CLI tool or with Jenkins pipelines.

Tere are two postgres services for local environmet:

    `postgres-live` - use bootstrap script to emulate a non-empty database. Listening port is ${.env.POSTGRES_PORT_LIVE} 
    `postgres-dev`  - boot as an empty database. Listening port is ${.env.POSTGRES_PORT_DEV} 


## Database Refactor Structure
----
All  schema changes have a nested structure and are grouped in a common file db.changelogs-root.xml file.

```
└── src
    ├── healthchecks
    │   └── postgres.sh
    └── main
        └── resources
            └── db
                ├── db.changelogs-root.xml
                ├── seed
                │   ├── city.csv
                │   ├── country.csv
                │   ├── countrylanguage.csv
                │   └── init_database_v1.sh
                ├── v1.0
                │   ├── YYYY-MM-DD-01-changelog.xml
                │   ├── YYYY-MM-DD-N-changelog.xml
                │   └── cumulative-changelog.xml
                ├── v2.0
                │   ├── YYYY-MM-DD-01-changelog.xml
                |   ├── YYYY-MM-DD-N-changelog.xml
                │   └── cumulative-changelog.xml
                └── v3.0
                    ├── YYYY-MM-DD-01-changelog.xml
                    ├── YYYY-MM-DD-N-changelog.xml
                    └── cumulative-changelog.xml
```

## Prerequisites
----
1. Install JDK 1.8
2. Docker v20.x+
3. Docker Compose v2.x+
4. Liquibase CLI v4.17+. How to do guide [here](https://docs.liquibase.com/start/install/home.html).
5. Jenkins lts v2.3x+


## Setup Local Enviroment
----
1. Jump to project dir: `cd ${PROJECT_PATH}`
2. Run postgres db servises: `docker-compose up -d`

    ```
    ❯ docker ps --format '{{ .Image }}\t{{ .Status }}\t{{ .Ports }}\t{{ .Names }}' | grep postgres-
        postgres:15-alpine	Up (healthy)	0.0.0.0:5432->5432/tcp	postgres-live
        postgres:15-alpine	Up (healthy)	0.0.0.0:5430->5432/tcp	postgres-dev
    ```
3. Add Jenkinsfile of the project to Jenkins server.


## Liquibase CLI profile preparation
----
1. Jump to project dir: `cd ${PROJECT_PATH}`
2. Create file _<env>.liquibase.properties_ with correct values.
    ```
    liquibase.hub.mode:off
    liquibase.searchPath: ./src/main/resources/db/
    changeLogFile: db.changelogs-root.xml
    driver: org.postgresql.Driver
    liquibase.secureParsing: false
    url: jdbc:postgresql://localhost:${.env.POSTGRES_PORT_DEV}/${.env.POSTGRES_DB}
    username: ${.env.POSTGRES_USER}
    password: ${.env.POSTGRES_PASSWORD}
    ```

## Cases Study
### Set up Liquibase CLI for connecting to the database with profile

    >touch dev.liquibase.properties
    >touch live.liquibase.properties

### Test connecting to `postgres-dev` and `postgres-live` 
    liquibase --defaults-file=dev.liquibase.properties status --contexts="dev"
    liquibase --defaults-file=live.liquibase.properties status --contexts="live"

### Update and rollback to specific version of db migration. Update up to __v1.2__ then rollback to __v1.1__
    liquibase --defaults-file=dev.liquibase.properties update-to-tag 1.2 --contexts="dev"
    liquibase --defaults-file=dev.liquibase.properties history
    liquibase --defaults-file=dev.liquibase.properties rollback --tag=1.1 --contexts="dev"
    liquibase --defaults-file=dev.liquibase.properties history

### Apply all new changes on `postgres-dev`. Update up to __v3.1__
    liquibase --defaults-file=dev.liquibase.properties update --contexts="dev"
    liquibase --defaults-file=dev.liquibase.properties status --contexts="dev"

### Sync already deploy changes and Add all new changes on `postgres-live`
    liquibase --defaults-file=live.liquibase.properties update --contexts="live"
    liquibase --defaults-file=dev.liquibase.properties history --contexts="live"



### Command examples:
----
Liquibase command syntax: `liquibase [GLOBAL OPTIONS] [COMMAND] [COMMAND OPTIONS]`

To validate the changelog for errors:

    liquibase --defaults-file=liquibase.properties validate

To execute dry-run:

	liquibase --defaults-file=liquibase.properties update-sql

To execute changesets:

	liquibase --defaults-file=liquibase.properties update

To tag a database based on the project pom version:

	liquibase --defaults-file=liquibase.properties tag ${TAG_NUMBER}

To rollback sqls till a tag version:

	liquibase --defaults-file=liquibase.properties rollback --tag=${TAG_NUMBER}

To list all deployed changesets and their deployment ID:

	liquibase --defaults-file=liquibase.properties history

To generates html documentation for the existing database and changelogs:

    liquibase --defaults-file=liquibase.properties db-doc changelogDocs


## Reference
----
- [Liquibase Project](https://github.com/liquibase/liquibase)
- [Liquibase Concepts](https://docs.liquibase.com/concepts/home.html)
- [Liquibase Parameters](https://docs.liquibase.com/parameters/home.html)
- [Liquibase Commands](https://docs.liquibase.com/commands/home.html)