# Requirements

## Backend requirements

* a webserver (e.g. Nginx, Apache)
* PHP >= 8.1
* PHP extensions:
    * DOM
    * PDO\_MYSQL (if you use MariaDB)
    * PDO\_PGSQL (if you use PostgreSQL)
    * mbstring
    * filter
* one of the following database:

  | Database       | Supported versions         |
  |----------------|----------------------------|
  | **MariaDB**        | 10.4 / 10.5 / 10.6 / 10.11 |
  | **PostgreSQL**     | 12 / 13 / 14 / 15          |

!!! warning "MySQL 8"
    We not recommand using MySQL 8.x. We had testing it and some queries (`WHERE IN`) return no data instead many data and it's really a problem. The problem appears with version 8.0.17 and for last version at 19 december 2023, the 8.0.34 version not fixed.

## Frontend requirements

* a webserver (e.g. Nginx, Apache)
* Node.js (tested with version 18)
