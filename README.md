# tbls [![Build Status](https://travis-ci.org/k1LoW/tbls.svg?branch=master)](https://travis-ci.org/k1LoW/tbls) [![GitHub release](https://img.shields.io/github/release/k1LoW/tbls.svg)](https://github.com/k1LoW/tbls/releases) [![codecov](https://codecov.io/gh/k1LoW/tbls/branch/master/graph/badge.svg)](https://codecov.io/gh/k1LoW/tbls)

`tbls` is a CI-Friendly tool for document a database, written in Go.

Key features of `tbls` are:

- **Document a database automatically in GitHub Friendly Markdown format**
- **Single binary**
- **CI-Friendly**
- **Work as linter for database document**

### Table of Contents

  - [Quick Start](#quick-start)
  - [Install](#install)
  - [Getting Started](#getting-started)
    - [Document a database](#document-a-database)
    - [Diff database and document](#diff-database-and-document)
    - [Lint a database](#lint-a-database)
    - [Continuous Integration](#continuous-integration)
  - [Support Database](#support-database)

<br>

## Quick Start

Document a database with one command.

``` console
$ tbls doc postgres://dbuser:dbpass@hostname:5432/dbname
```

## Install

**homebrew tap:**

```console
$ brew install k1LoW/tap/tbls
```

**manually:**

Download binany from [releases page](https://github.com/k1LoW/tbls/releases)

**go get:**

```console
$ go get github.com/k1LoW/tbls
```

## Getting Started

### Document a database

Add `.tbls.yml` file to your repogitory.

``` yaml
# .tbls.yml

# DSN (Databaase Source Name) to connect database
dsn: postgres://dbuser:dbpass@localhost:5432/dbname

# Path to generate document
# Default is `dbdoc`
docPath: doc/schema
```

> **Notice:** If you are using a symbol such as `#` `<` in database password, URL-encode the password

Run `tbls doc` to analyzes the database and generate document in GitHub Friendly Markdown format.

``` console
$ tbls doc
```

> **Tips:** If you can use Graphviz `dot` command, `tbls doc` generate ER diagram images at the same time.

Commit `.tbls.yml` and the document.

``` console
$ git add .tbls.yml doc/schema
$ git commit -m'Add database document'
$ git push origin master
```

View the document on GitHub.

[Sample document](sample/postgres/README.md)

![sample](sample/doc.png)

### Diff database and document

Update database schema.

``` console
$ psql -U dbuser -d dbname -h hostname -p 5432 -c 'ALTER TABLE users ADD COLUMN phone_number varchar(15);'
Password for user dbuser:
ALTER TABLE
```

`tbls diff` shows the difference between database schema and generated document.

``` diff
$ tbls diff
diff postgres://dbuser:*****@hostname:5432/dbname doc/schema/README.md
--- postgres://dbuser:*****@hostname:5432/dbname
+++ doc/schema/README.md
@@ -4,7 +4,7 @@

 | Name | Columns | Comment | Type |
 | ---- | ------- | ------- | ---- |
-| [users](users.md) | 7 | Users table | BASE TABLE |
+| [users](users.md) | 6 | Users table | BASE TABLE |
 | [user_options](user_options.md) | 4 | User options table | BASE TABLE |
 | [posts](posts.md) | 8 | Posts table | BASE TABLE |
 | [comments](comments.md) | 6 | Comments<br>Multi-line<br>table<br>comment | BASE TABLE |
diff postgres://dbuser:*****@hostname:5432/dbname doc/schema/users.md
--- postgres://dbuser:*****@hostname:5432/dbname
+++ doc/schema/users.md
@@ -14,7 +14,6 @@
 | email | varchar(355) |  | false |  |  | ex. user@example.com |
 | created | timestamp without time zone |  | false |  |  |  |
 | updated | timestamp without time zone |  | true |  |  |  |
-| phone_number | varchar(15) |  | true |  |  |  |

 ## Constraints

```

> **Notice:** `tbls diff` shows the difference Markdown documents only.

### Lint a database

Add linting rule to `.tbls.yml` following

``` yaml
# .tbls.yml
lint:
  requireColumnComment:
    enabled: true
    exclude:
      - id
      - created
      - updated
  columnCount:
    enabled: true
    max: 10
```

Run `tbls lint` to check the database according to `lint:` rules

``` console
$ tbls lint
users.username: column comment required.
users.password: column comment required.
users.phone_number: column comment required.
posts.user_id: column comment required.
posts.title: column comment required.
posts.labels: column comment required.
comments.post_id: column comment required.
comment_stars.user_id: column comment required.
post_comments.comment: column comment required.
posts has too many columns. [12/10]
comments has too many columns. [11/10]
```

### Continuous Integration

Continuous integration using tbls.

1. Commit the document using `tbls doc`.
2. Update the database schema in the development cycle.
3. Check for document updates by running `tbls diff` or `tbls lint` in CI.
4. Return to **1**.

**Example: Travis CI**

``` yaml
# .travis.yml
language: go

install:
  - source <(curl -sL https://git.io/use-tbls)
script:
  - tbls diff
  - tbls lint
```

> **Tips:** If your CI based on Debian/Ubuntu (`/bin/sh -> dash`), you can use following install command `curl -sL https://git.io/use-tbls > use-tbls.tmp && . ./use-tbls.tmp && rm ./use-tbls.tmp`

> **Tips:** If the order of the columns does not match, you can use the `--sort` option.

## Configration

### DSN

`DSN` (Data Srouce Name) is used to connect to database.

``` yaml
# .tbls.yml
dsn: my://dbuser:dbpass@hostname:3306/dbname
```

`DSN` can expand environment variables.

``` yaml
# .tbls.yml
dsn: my://${MYSQL_USER}:${MYSQL_PASSWORD}@localhost:3306/${MYSQL_DATABASE}
```

#### Support Database

tbls support following databases.

**PostgreSQL:**

``` yaml
# .tbls.yml
dsn: postgres://dbuser:dbpass@hostname:5432/dbname
```

``` yaml
# .tbls.yml
dsn: pg://dbuser:dbpass@hostname:5432/dbname
```

**MySQL:**

``` yaml
# .tbls.yml
dsn: mysql://dbuser:dbpass@hostname:3306/dbname
```

``` yaml
# .tbls.yml
dsn: my://dbuser:dbpass@hostname:3306/dbname
```

**SQLite:**

``` yaml
# .tbls.yml
dsn: sqlite:///path/to/dbname.db
```

``` yaml
# .tbls.yml
dsn: sq:///path/to/dbname.db
```

### Document path

### Lint

### Comments

### Relations




### 1. Command arguments

``` console
$ tbls doc my://root:mypass@localhost:33306/testdb sample/mysql
```

### 2. Use `.tbls.yml` or set `--config` option

Put `.tbls.yml` on execute directory or specify with the `--config` option.

YAML format is follows

``` yaml
---
dsn: my://root:mypass@localhost:33306/testdb
docPath: sample/mysql
```

``` yaml
---
dsn: my://${MYSQL_USER}:${MYSQL_PASSWORD}@localhost:33306/${MYSQL_DATABASE}
docPath: sample/mysql
```

### 3. Envirionment

``` console
$ env TBLS_DSN=my://root:mypass@localhost:33306/testdb TBLS_DOC_PATH=sample/mysql tbls doc
```

## Add additional data (relations, comments) to schema

To add additional data to the schema, add settings to `.tbls.yml` or `--config` like [YAML file](testdata/additional_data.yml) (`relations`, `comments`)
