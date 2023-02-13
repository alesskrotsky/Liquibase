# Liquibase
Database Migrations with Liquibase
This project demonstrates database refactoring using Liquibase commands against postgres database. Here are two options for using Liquibase as a CLI tool or with Jenkins pipelines.

Tere are two postgres services for local environmet:

    `postgres-live` - use bootstrap script to emulate a non-empty database. Listening port is ${.env.POSTGRES_PORT_LIVE} 
    `postgres-dev`  - boot as an empty database. Listening port is ${.env.POSTGRES_PORT_DEV} 

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
    ❯ docker ps --format '{{ .Image }}\t{{ .Status }}\t{{ .Ports }}\t{{ .Names }}'
        postgres:15-alpine	Up (healthy)	0.0.0.0:5432->5432/tcp	postgres-live
        postgres:15-alpine	Up (healthy)	0.0.0.0:5430->5432/tcp	postgres-dev
    ```
3. Add Jenkinsfile of the project to Jenkins server.


## Liquibase CLI profile preparation
----
1. Jump to project dir: `cd ${PROJECT_PATH}`
2. Create file _local.liquibase.properties_ with correct values.
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


### Command examples:
----
To validate the changelog for errors:

    liquibase --defaults-file=local.liquibase.properties validate

To execute dry-run:

	liquibase --defaults-file=local.liquibase.properties update-sql

To execute changesets:

	liquibase --defaults-file=local.liquibase.properties update

To tag a database based on the project pom version:

	liquibase --defaults-file=local.liquibase.properties tag ${TAG_NUMBER}

To rollback sqls till a tag version:

	liquibase --defaults-file=local.liquibase.properties rollback --tag=${TAG_NUMBER}


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
                │   └── init_database_v1.sh
                ├── v0.1
                │   ├── YYYY-MM-DD-01-changelog.xml
                │   ├── YYYY-MM-DD-N-changelog.xml
                │   └── cumulative-changelog.xml
                ├── v0.2
                │   ├── YYYY-MM-DD-01-changelog.xml
                |   ├── YYYY-MM-DD-N-changelog.xml
                │   └── cumulative-changelog.xml
                └── v0.3
                    ├── YYYY-MM-DD-01-changelog.xml
                    ├── YYYY-MM-DD-N-changelog.xml
                    └── cumulative-changelog.xml
```


## Cases Study
- Setting up and connecting to the database using Liquibase CLI
- Sync already deploy changes on `postgres-live` 
- Update and rollback to specific tag 
- Generate a changeset execution report
- Add and rollout new changes


## Reference
----
- [Liquibase Project](https://github.com/liquibase/liquibase)
- [Liquibase Concepts](https://docs.liquibase.com/concepts/home.html)
- [Liquibase Parameters](https://docs.liquibase.com/parameters/home.html)
- [Liquibase Commands](https://docs.liquibase.com/commands/home.html)