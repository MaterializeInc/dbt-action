# Materialize dbt action

A GitHub action that runs dbt commands against a Materialize instance.

## Required secrets

The following secrets are required:

| Name | Description | Example |
| ---- | ----------- | ------- |
| `MZ_PASS` | The password for the Materialize user. | `MZ_PASS: ${{ secrets.MZ_PASS }}` |
| `MZ_USER` | The username for the Materialize user. | `MZ_USER: ${{ secrets.MZ_USER }}` |
| `MZ_HOST` | The host for the Materialize instance. | `MZ_HOST: ${{ secrets.MZ_HOST }}` |

## Variables

You can override the default values for the following variables:


| Name                 | Description                                  | Default       |
| -------------------- | -------------------------------------------- | ------------- |
| `DBT_MZ_VERSION`     | `dbt-materialize` adapter version to install | `latest`      |
| `SKIP_INSTALL` | If the action should skip installing dbt | |
| `MZ_DATABASE`        | The database for the Materialize instance    | `materialize` |
| `MZ_SCHEMA`          | The schema for the Materialize instance      | `public`      |
| `MZ_CLUSTER`         | The cluster for the Materialize instance     | `default`     |
| `DBT_PROJECT_FOLDER` | The dbt project folder                       | `.`           |

## Usage

Create a new workflow file in your repository, for example `.github/workflows/materialize-dbt-action.yml` and add the following content:

```yaml
name: Materialize dbt action
on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0
    - name: Materialize dbt action
      uses: MaterializeInc/dbt-action@main
      with:
        dbt_command: "dbt --version --profiles-dir /"
      env:
        DBT_MZ_VERSION: 1.3.4
        MZ_PASS: ${{ secrets.MZ_PASS }}
        MZ_USER: ${{ secrets.MZ_USER }}
        MZ_HOST: ${{ secrets.MZ_HOST }}
        MZ_DATABASE: materialize
        MZ_SCHEMA: public
        MZ_CLUSTER: default
        DBT_PROJECT_FOLDER: ${{ env.DBT_PROJECT_FOLDER }}
```

The above will run the `dbt --version` command against your Materialize instance on every push to the `main` branch.

To run on a pull request, change the `on` section to:

```yaml
on:
  pull_request:
    branches:
      - main
```

## Inputs

### `dbt_command`

**Required** The dbt command to run. Default `"dbt debug"`. You can change this to run any dbt command.

## Overriding the default `profiles.yml`

You can override the default `profiles.yml` by creating a new file in your repository called `profiles.yml` and add the following content:

```yaml
default:
  outputs:
    default:
      type: materialize
      threads: 1
      user: "{{ env_var('MZ_USER')}}"
      pass: "{{ env_var('MZ_PASS')}}"
      host: "{{ env_var('MZ_HOST')}}"
      port: 6875
      database: materialize
      schema: public
      cluster: default
      sslmode: require

  target: default
```

Then change the `dbt_command` input to `dbt --version --profiles-dir .` if you have a `profiles.yml` file in your repository. The `${DBT_PROJECT_FOLDER}` will be added to the `--profiles-dir` argument.

This will override the default `profiles.yml` file and use the one from your repository.

## `DBT_PROJECT_FOLDER`

You can specify a custom dbt project folder by setting the `DBT_PROJECT_FOLDER` environment variable. This is useful if you have multiple dbt projects in the same repository.


## `dbt-materialize` version

By default the action will install the latest version of the `dbt-materialize` adapter.
You can specify a specific version by setting the `DBT_MZ_VERSION` environment variable. For example:

```yaml
env:
  DBT_MZ_VERSION: 1.3.2
```

## Usage with Poetry
If you are using [poetry](https://python-poetry.org/) to manage your dbt dependency, you can choose to skip in the installation of the dbt version with this action and install your necessary dependencies separately.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 0
    - uses: actions/setup-python@v4
      with:
        python-version: 3.9
    - name: Run image
      uses: abatilo/actions-poetry@v2
      with:
        poetry-version: 1.0
    - name: Poetry install
      run: poetry config virtualenvs.create false && poetry install --no-interaction --no-dev --no-ansi
    - name: Materialize dbt action
      uses: MaterializeInc/dbt-action@main
      with:
        dbt_command: "dbt --version --profiles-dir /"
      env:
        SKIP_INSTALL: true
        MZ_PASS: ${{ secrets.MZ_PASS }}
        MZ_USER: ${{ secrets.MZ_USER }}
        MZ_HOST: ${{ secrets.MZ_HOST }}
        MZ_DATABASE: materialize
        MZ_SCHEMA: public
        MZ_CLUSTER: default
        DBT_PROJECT_FOLDER: ${{ env.DBT_PROJECT_FOLDER }}
```