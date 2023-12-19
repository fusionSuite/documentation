# Getting started

This documentation is intended for the developers of FusionSuite.

FusionSuite code is separated in two repositories:

- [github.com/fusionSuite/backend](https://github.com/fusionSuite/backend)
- [github.com/fusionSuite/frontend](https://github.com/fusionSuite/frontend)

The backend is written in PHP (>= 8.1). It uses the micro-framework [Slim](https://www.slimframework.com/)
and the [Eloquent](https://laravel.com/docs/9.x/eloquent) ORM. The backend
provides a REST API which is used by the frontend.

Three databases are supported by the backend: MySQL, MariaDB and PostgreSQL.

The frontend is written in TypeScript. It uses the framework [Angular](https://angular.io/).

There are tests mainly for the REST API, but there are some for the frontend as
well.
