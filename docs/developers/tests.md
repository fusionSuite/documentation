# Execution of the tests & linters

## Backend

The backend test suite cannot be easily executed locally yet.

!!! info
    The issue is that we are testing the REST API at a specific address which
    cannot be configured yet. **More information on Github:
    [fusionSuite/FusionSuite#5](https://github.com/fusionSuite/FusionSuite/issues/5).**

## Frontend

There are two kind of tests for the frontend: unit tests and end-to-end tests.

!!! warning
    Due to the complexity of the setup, none of these tests can be executed
    with Docker. You’ll have to install the frontend dependencies locally.

### Unit tests

Unit tests are based on the default [Angular testing](https://angular.io/guide/testing)
setup.

To execute them:

```console
$ make test
```

It will look for a Chrome binary. If you’ve installed Chrome in a specific
location, or if you use Chromium, you may have to pass the `CHROME_BIN`
environment variable:

```console
$ make test CHROME_BIN=/usr/bin/chromium-browser
```

### End-to-end tests

End-to-end (e2e) tests are based on [Cypress](https://cypress.io).

!!! warning
    **These tests are destructive.** If you don’t want to lose your development
    data, you should consider to create a dedicated backend environment for your
    e2e tests. For instance with:

    ```console
    $ make setup ENV_NAME=e2e DB_NAME=fusionsuite_e2e
    ```

    Obviously, you’ll have first to create the `fusionsuite_e2e` database
    before and give the correct permissions to the `fusionsuite` user.

To execute them:

```console
$ make e2e-open
```

It will open a window to allow you to select a browser in which the tests will
be executed. Once the brower is opened, just click on the spec file that you
want to execute.

The tests need to communicate with the backend at the system level to reset the
database before each test. Cypress expects to find the backend at `../backend`.
If you’ve placed the backend at a different location, you can pass the
`cypress_backend_path` variable:

```console
$ make e2e-open cypress_backend_path=../fusionsuite-backend/
```

You may also need to precise the backend URL with the `cypress_backend_server`
(default is `http://localhost:8000`). For instance:

```console
$ make e2e-open cypress_backend_server=http://localhost/backend
```

## Linters

To execute the linters (backend or frontend), run the following command:

```console
$ make lint
```

You also can automatically fix some problems raised by the linter with:

```console
$ make lint-fix
```

## Continuous Integration (CI)

All the test suites and linters are automatically executed when commits are
pushed on the `master` branches and on pull requests.

You’ll have to make sure that all the checks are green before being able to
merge on `master`.

You can find the history of CI at:

- [github.com/fusionSuite/backend/actions](https://github.com/fusionSuite/backend/actions)
- [github.com/fusionSuite/frontend/actions](https://github.com/fusionSuite/frontend/actions)
