# Brooklyn Data Co. dbt coding conventions

  - [General guidelines](#general-guidelines)
  - [Profiles](#profiles)
  - [Projects](#projects)
    - [Files](#files)
    - [dbt_project.yml files](#dbt_projectyml-files)
  - [Packages](#packages)
  - [Sources](#sources)
  - [Models](#models)
    - [Schemas](#schemas)
    - [schema.yml files](#schemayml-files)
  - [Seeds](#seeds)
  - [Macros](#macros)
  - [YAML](#yaml)
  - [Credits](#credits)

<br>

## General guidelines

#### Follow [our SQL style guide](sql_style_guide.md).
Learn it, know it, live it. 🏄

<br>

#### Consider the [official dbt best practices](https://docs.getdbt.com/docs/best-practices).
One of which is to use coding conventions and a style guide. 👍

<br>

#### Optimize primarily for readability, maintainability, and robustness rather than for fewer lines of code.
Newlines are cheap; people's time is expensive.

<br>

#### Lines should ideally not be longer than 120 characters.
Very long lines are harder to read, especially in situations where space may be limited like on smaller screens or in side-by-side version control diffs.

<br>

#### Identifiers such as model names and macros should be in lowercase `snake_case`.
It's more readable, and easier to keep consistent.

<br>

## Profiles

#### The `profiles.yml` file should only be accessible to the person it belongs to.
It contains database credentials, so file permissions should be set to restrict access to it.

<br>

#### Profile names should include the name of the company they're for.
To avoid naming conflicts when we have multiple profiles.

<br>

#### Production targets should be named `prod`.
dbt's built-in `generate_schema_name_for_env` macro assumes production targets are named `prod`.

<br>

#### Production targets should not be set as the default in developers' profiles.
Developers should target a dev environment by default.

<br>

## Projects

### Files

For consistency (especially with version control):
  - Files should be encoded as UTF-8.
  - Indentation should always be done with space characters, never with tab characters.
  - Files should have Unix-style endlines (i.e. LF, not CRLF).
  - All trailing whitespace should be stripped from files.
  - Files should always end with a newline.

<br>

#### Projects should contain a `.editorconfig` file to help keep file formats consistent.
If you use Atom install the [`editorconfig` package](https://atom.io/packages/editorconfig) (support is also available for [other editors](https://editorconfig.org/#download)).

```editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.yml]
indent_size = 2
```

<br>

#### Projects should contain a `.gitignore` file to avoid committing extraneous files to version control.
For example:
  - dbt's compilation artifacts, downloaded package dependencies, and logs.
  - `.DS_Store` files on Macs.
  - `desktop.ini` files on Windows.

```gitignore
target/
dbt_modules/
logs/

.*
!.editorconfig
!.git?*

desktop.ini
```

<br>

### `dbt_project.yml` files

#### Make `ephemeral` the default materialization.
Concrete materializations should be opt-in.

```yaml
models:
  materialized: ephemeral
```

<br>

#### When targeting Redshift, enable [late binding views](https://docs.getdbt.com/docs/redshift-configs#section-late-binding-views) globally.
This prevents views from getting dropped prematurely when their source models get rebuilt.

```yaml
models:
  bind: false  # Materialize all views as late-binding.
```

<br>

#### Don't apply a global `enabled` config setting to all models.
It shouldn't be necessary (models are enabled by default), and it will override any `enabled` config settings within packages which can cause problems.

```yaml
models:
  enabled: true  # Bad
```

<br>

## Packages

#### Use packages from [dbt Hub](https://hub.getdbt.com/) whenever possible.
This allows dbt to handle duplicate dependencies.

```yaml
# Good
packages:
  - package: fishtown-analytics/dbt_utils

# Bad
packages:
  - git: https://github.com/fishtown-analytics/dbt-utils.git
```

<br>

#### Pin packages to the latest patch version from a specific minor release.
This allows bug fixes to be picked up automatically while avoiding any unexpected functional changes.

```yaml
# Good
packages:
  - package: fishtown-analytics/dbt_utils
    version: [">=0.1.0", "<0.2.0"]

# Bad
packages:
  - package: fishtown-analytics/dbt_utils
    version: 0.1.17

# Bad
packages:
  - git: https://github.com/fishtown-analytics/dbt-utils.git
    revision: master
```

<br>

## Sources

#### Each [dbt source](https://docs.getdbt.com/docs/using-sources) should be defined in its own `schema.yml` file named `<source name>.yml`.
Sources have some nice features:
  - Showing the source table schemas in the docs.
  - Being able to easily run all models that select from a particular source.
  - Testing source columns.
  - Calculating the freshness of source data.

<br>

## Models

#### Model naming conventions:
  - Model names should be plurals (tables are collections of multiple things).
  - Source models should be named `<source name>__<table name>` (two underscores between the source name and table name because those names may contain underscores themselves).
  - Staging model names should be prefixed with `stage_`.
  - Dimension model names should be prefixed with `dim_`.
  - Fact model names should be prefixed with `fact_`.
  - Aggregate model names should be prefixed with `agg_`.
  - Bridge model names should be prefixed with `bridge_`.

<br>

#### Only source models should select from sources.
  - Source models should select from [dbt sources](https://docs.getdbt.com/docs/using-sources), not directly from the source tables themselves.
  - All other models should only select from other models.

```sql
/* Good */
select ...
from {{ source('web', 'pageview') }}

/* Bad */
select ...
from web.pageview
```

<br>

#### Source models should alias source columns as necessary to conform to our naming conventions.
For example, source column names that conflict with reserved words must be aliased to a different name.

<br>

#### Most models should have a single primary key column.
This makes joins easier and more performant.

  - Instead of having a composite (multi-column) primary key, consider creating a surrogate primary key.
  - Aggregate and bridge models are exceptions, as they often don't need normal primary keys.

<br>

#### Model-specific attributes should be specified directly in the model.
For example, sort and dist keys for Redshift:
```sql
{{
config(
    materialized='table'
    , sort='created_at'
    , dist='order_id'
)
}}
```

<br>

#### When using Jinja delimiters, put spaces on the inside of the delimiters.
```sql
/* Good */
{{ ref('customers') }}

/* Bad */
{{ref('customers')}}

/* Good */
{% if where_clause is not none %}

/* Bad */
{%if where_clause is not none%}
```

<br>

#### Put no spaces around the equals sign for macro keyword arguments.
Like in Python.

```sql
/* Good */
{{ foo('bar', baz=123) }}

/* Bad */
{{ foo('bar', baz = 123) }}
```

<br>

### Schemas

#### Use [dbt's alternative pattern for handling custom schemas](https://docs.getdbt.com/docs/using-custom-schemas#section-an-alternative-pattern-for-generating-schema-names).
  - In production, custom schemas are used verbatim (not concatenated with the target schema name).
  - In all other environments, custom schemas are ignored (all models go in the single target schema).

```sql
/* Put this in `macros/generate_schema_name.sql`. */

{% macro generate_schema_name(custom_schema_name, node) -%}
    {{ generate_schema_name_for_env(custom_schema_name, node) }}
{%- endmacro %}
```

<br>

#### Materialized source models and staging models should go in a separate schema named `<production target schema>_staging`.
Such models should not be used directly in normal analysis/reporting, and putting them in a separate "staging" schema helps enforce that.

<br>

#### Models containing especially sensitive data should go in a separate schema named `<production target schema>_sensitive`.
Permissions should be set for the "sensitive" schema to restrict access as appropriate.

<br>

### `schema.yml` files

#### Documentation and tests for each model should be placed in a `schema.yml` file alongside the model file named `<model name>.yml`.
Having a separate `schema.yml` file for each model has several benefits:
  - Makes it easier to find the documentation and tests for a model.
  - Clearly shows which models have documentation/tests and which don't.
  - Helps avoid version control merge conflicts.

<br>

#### A model is not complete without tests and documentation.
Testing and documenting models should be an integral part of their development.

<br>

#### Columns should be listed in `schema.yml` files in the same order they appear in their model.
This makes it easier to correlate between the model and the `schema.yml` file.

Also, if comments are used in the model to label groupings of columns, consider putting matching comments in the `schema.yml` file.

<br>

#### Prefer Markdown for formatting model and column descriptions instead of HTML.
Markdown is easier to read and edit.

One exception is line breaks, where using `<br>` is preferable to hassling with Markdown's line break syntax of ending a line with two or more spaces (which is almost impossible to notice and extremely easy to accidentally break).

```yaml
# Good
models:
  - name: fact_orders
    description: |
      **Overview:** Summary data for orders.
      <br>**Data sources:** `orders`, `order_line_items`

# Bad
models:
  - name: fact_orders
    description: |
      <b>Overview:</b> Summary data for orders.
      <br><b>Data sources:</b> <code>orders</code>, <code>order_line_items</code>
```

<br>

#### Column tests:
  - `unique` and `not_null` tests should be applied to the primary key.
  - `relationships` tests should be applied to the foreign keys.

<br>

## Seeds

#### Seed file names should be prefixed with `seed_`.
The file name is used as the table name.

<br>

#### Seed tables should go in a separate schema named `<production target schema>_staging`.
If seed data needs to be used directly in analysis/reporting then a model should be created to select from the seed data and apply any formatting or other safeguards, and then that model can be materialized in the default target schema.

<br>

#### Beware of dbt auto-formatting the seed data.
It appears dbt always does Excel-like data type detection and auto-formatting, and even [overriding the column types](https://docs.getdbt.com/docs/seeds#section-override-column-types) doesn't prevent that.  For example, when overriding a seed column for 5-digit zip codes to be `varchar(5)` a value in the file like `00123` will still end up being `123` in the resulting seed table.

So until dbt fixes the issue with auto-formatting seed data you may need to add a dummy row with non-numeric values to trick dbt into not auto-formatting such data.

<br>

## Macros

#### Macros should be defined in their own file named `<macro name>.sql`.
This makes it easier to locate macros in the project.

One exception is [adapter-specific macros](https://docs.getdbt.com/docs/building-a-new-adapter#section-adapter-macros), which should go in the same file as their associated multiple-dispatch macro.

<br>

## YAML

  - Indents should be 2 spaces.
  - List items should be indented.
  - Use a new line to separate list items that are dictionaries where appropriate.

```yaml
models:
  - name: events
    columns:
      - name: event_id
        description: "This is a unique identifier for the event."
        tests:
          - unique
          - not_null

      - name: event_time
        description: "When the event occurred in UTC (eg. 2018-01-01 12:00:00)."
        tests:
          - not_null

      - name: user_id
        description: "The ID of the user who recorded the event."
        tests:
          - not_null
          - relationships:
              to: ref('users')
              field: id
```

<br>

#### Values of expository text fields should always be double-quoted or be block scalars.
Otherwise it's too easy to accidentally break the YAML syntax by unwittingly using a character sequence YAML considers special.

```yaml
# Good
models:
  - name: foo
    columns:
      - name: bar
        description: "A colon: followed by a space and a #hashmark following a space are allowed in double-quoted flow scalars."

      - name: baz
        description: |
          A colon: followed by a space
          and a #hashmark following a space
          are allowed in block scalars.

# Bad
models:
  - name: foo
    columns:
      - name: bar
        description: A colon: followed by a space and a #hashmark following a space are invalid in plain flow scalars.

      - name: baz
        description:
          A colon: followed by a space
          and a #hashmark following a space
          are invalid in plain flow scalars.
```

<br>

## Credits

These coding conventions were inspired in part by:
  - [Fishtown Analytics' dbt coding conventions](https://github.com/fishtown-analytics/corp/blob/b5c6f55b9e7594e1a1e562edf2378b6dd78a1119/dbt_coding_conventions.md)
  - [GitLab's SQL style guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)
