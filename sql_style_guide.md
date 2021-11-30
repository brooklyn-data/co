# Brooklyn Data Co. SQL style guide

  - [General guidelines](#general-guidelines)
  - [Syntax](#syntax)
    - [Joins](#joins)
    - [CTEs](#ctes)
  - [Naming](#naming)
  - [Formatting](#formatting)
  - [Credits](#credits)

<br>

## General guidelines

#### Optimize primarily for readability, maintainability, and robustness rather than for fewer lines of code.
Newlines are cheap; people's time is expensive.

<br>

#### Avoid having enormous `select` statements.
If a `select` statement is so large it can't be easily comprehended, it would be better to refactor it into multiple smaller CTEs that are later joined back together.

<br>

#### Lines should ideally not be longer than 120 characters.
Very long lines are harder to read, especially in situations where space may be limited like on smaller screens or in side-by-side version control diffs.

<br>

#### Identifiers such as aliases and CTE names should be in lowercase `snake_case`.
It's more readable, easier to keep consistent, and avoids having to quote identifiers due to capitalization, spaces, or other special characters.

<br>

#### Never use reserved words as identifiers.
Otherwise the identifier will have to be quoted everywhere it's used.

<br>

#### Never use tab characters.
It's easier to keep things consistent in version control when only space characters are used.

<br>

## Syntax

#### Keywords and function names should all be lowercase.
Lowercase is more readable than uppercase, and you won't have to constantly be holding down a shift key.

```sql
/* Good */
select *
from customers

/* Good */
select count(*) as customers_count
from customers

/* Bad */
SELECT *
FROM customers

/* Bad */
Select *
From customers

/* Bad */
select COUNT(*) as customers_count
from customers
```

<br>

#### Use `!=` instead of `<>`.
`!=` reads like "not equal" which is closer to how we'd say it out loud.

<br>

#### Use `||` instead of `concat`.
`||` is a standard SQL operator, and in some databases like Redshift `concat` only accepts two arguments.

<br>

#### Use `coalesce` instead of `ifnull` or `nvl`.
  - `coalesce` is universally supported, whereas Redshift doesn't support `ifnull` and BigQuery doesn't support `nvl`.
  - `coalesce` is more flexible and accepts an arbitrary number of arguments.

<br>

#### Use `is null` instead of `isnull`, and `is not null` instead of `notnull`.
`isnull` and `notnull` are specific to Redshift.

<br>

#### Use a `case` statement instead of `iff` or `if`.
`case` statements are universally supported, whereas Redshift doesn't support `iff`, and in BigQuery the function is named `if` instead of `iff`.

<br>

#### Always use the `as` keyword when aliasing columns, expressions, and tables.
```sql
/* Good */
select count(*) as customers_count
from customers

/* Bad */
select count(*) customers_count
from customers
```

<br>

#### Always alias grouping aggregates and other column expressions.
```sql
/* Good */
select max(id) as max_customer_id
from customers

/* Bad */
select max(id)
from customers
```

<br>

#### Use `where` instead of `having` when either would suffice.
Queries filter on the `where` clause earlier in their processing, so `where` filters are more performant.

<br>

#### Use `union all` instead of `union` unless duplicate rows really do need to be removed.
`union all` is more performant because it doesn't have to sort and de-duplicate the rows.

<br>

#### Use `select distinct` instead of grouping by all columns.
This makes the intention clear.

```sql
/* Good */
select distinct
    customer_id
    , date_trunc('day', created_at) as purchase_date
from orders

/* Bad */
select
    customer_id
    , date_trunc('day', created_at) as purchase_date
from orders
group by 1, 2
```

<br>

#### Avoid using an `order by` clause unless it's necessary to produce the correct result.
There's no need to incur the performance hit.  If consumers of the query need the results ordered they can normally do that themselves.

<br>

#### For functions that take date part parameters, specify the date part as a string rather than a keyword.
  - While some advanced SQL editors can helpfully auto-complete and validate date part keywords, if they get it wrong they'll show superfluous errors.
  - Less advanced SQL editors won't syntax highlight date part keywords, so using strings helps them stand out.
  - Using a string makes it unambiguous that it's not a column reference.

```sql
/* Good */
date_trunc('month', created_at)

/* Bad */
date_trunc(month, created_at)
```

<br>

#### Always use `/* */` comment syntax.
This allows single-line comments to naturally expand into multi-line comments without having to change their syntax.

When expanding a comment into multiple lines:
  - Keep the opening `/*` on the same line as the first comment text and the closing `*/` on the same line as the last comment text.
  - Indent subsequent lines by 4 spaces, and add an extra space before the first comment text to align it with the text on subsequent lines.

```sql
/* Good */

-- Bad

/*  Good:  Lorem ipsum dolor sit amet, consectetur adipiscing elit,
    sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
    Dolor sed viverra ipsum nunc aliquet bibendum enim. */

/* Bad:  Lorem ipsum dolor sit amet, consectetur adipiscing elit,
sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
Dolor sed viverra ipsum nunc aliquet bibendum enim. */

-- Bad:  Lorem ipsum dolor sit amet, consectetur adipiscing elit,
-- sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
-- Dolor sed viverra ipsum nunc aliquet bibendum enim.
```

<br>

#### Use single quotes for strings.
Some SQL dialects like BigQuery support using double quotes or even triple quotes for strings, but for most dialects:
  - Double quoted strings represent identifiers.
  - Triple quoted strings will be interpreted like the value itself contains leading and trailing single quotes.

```sql
/* Good */
select *
from customers
where email like '%@domain.com'

/* Bad */
select *
from customers
where email like "%@domain.com"
/* Will probably result in an error like `column "%@domain.com" does not exist`. */

/* Bad */
select *
from customers
where email like '''%@domain.com'''
/* Will probably be interpreted like '\'%domain.com\''. */
```

<br>

### Joins

#### Don't use `using` in joins.
  - Having all joins use `on` is more consistent.
  - If additional join conditions need to be added later, `on` is easier to adapt.
  - `using` can produce inconsistent results with outer joins in some databases.

<br>

#### Use `inner join` instead of just `join`.
It's better to be explicit so that the join type is crystal clear.

```sql
/* Good */
select *
from customers
inner join orders on customers.id = orders.customer_id

/* Bad */
select *
from customers
join orders on customers.id = orders.customer_id
```

<br>

#### In join conditions, put the table that was referenced first immediately after `on`.
This makes it easier to determine if the join is going to cause the results to fan out.

```sql
/* Good */
select *
from customers
left join orders on customers.id = orders.customer_id
/* primary key = foreign key --> one-to-many --> fan out */

/* Good */
select *
from orders
left join customers on orders.customer_id = customers.id
/* foreign key = primary key --> many-to-one --> no fan out */

/* Bad */
select *
from customers
left join orders on orders.customer_id = customers.id
```

<br>

#### When joining multiple tables, always prefix the column names with the table name/alias.
You should be able to tell at a glance where a column is coming from.

```sql
/* Good */
select
    customers.email
    , orders.invoice_number
    , orders.total_amount
from customers
inner join orders on customers.id = orders.customer_id

/* Bad */
select
    email
    , invoice_number
    , total_amount
from customers
inner join orders on customers.id = orders.customer_id
```

<br>

#### When inner joining, put filter conditions in the `where` clause instead of the `join` clause.
Only join conditions should be put in a `join` clause. All filter conditions should be put together in the `where` clause.

```sql
/* Good */
select
    ...
from orders
inner join customers on orders.customer_id = customers.id
where
    orders.total_amount >= 100
    and customers.email like '%@domain.com'

/* Bad */
select
    ...
from orders
inner join customers on
    orders.customer_id = customers.id
    and customers.email like '%@domain.com'
where orders.total_amount >= 100
```

<br>

### CTEs

  - Where performance permits, CTEs should perform a single, logical unit of work.
  - CTE names should be as verbose as needed to convey what they do.
  - CTE names should not be prefixed or suffixed with `cte`.
  - CTEs with confusing or notable logic should be commented.

<br>

#### Use CTEs rather than subqueries.
CTEs will make your queries more straightforward to read/reason about, can be referenced multiple times, and are easier to adapt/refactor later.

```sql
/* Good */
with
    paying_customers as (
        select *
        from customers
        where plan_name != 'free'
    )

select ...
from paying_customers

/* Bad */
select ...
from (
    select *
    from customers
    where plan_name != 'free'
) as paying_customers
```

<br>

## Naming

#### Name single-column primary keys `id`.
This allows us to easily tell at a glance whether a column is a primary key, helps us [discern whether joins are one-to-many or many-to-one](#in-join-conditions-put-the-table-that-was-referenced-first-immediately-after-on), and is more succinct than other primary key naming conventions (particularly in join conditions).

```sql
/* Good */
select ...
from orders
left join customers on orders.customer_id = customers.id
/* Easier to tell this is a many-to-one join and thus won't fan out. */

/* Bad */
select ...
from orders
left join customers on orders.customer_id = customers.customer_id

/* Good */
select ...
from orders
left join some_exceedingly_long_name on orders.some_exceedingly_long_name_id = some_exceedingly_long_name.id

/* Bad */
select ...
from orders
left join some_exceedingly_long_name on orders.some_exceedingly_long_name_id = some_exceedingly_long_name.some_exceedingly_long_name_id
```

<br>

#### Date/time column names:
  - Date columns based on UTC should be named like `<event>_date`.
  - Date columns based on a specific timezone should be named like `<event>_date_<timezone indicator>` (e.g. `order_date_et`).
  - Date+time columns based on UTC should be named like `<event>_at`.
  - Date+time columns based on a specific timezone should be named like `<event>_at_<timezone indicator>` (e.g `created_at_pt`).
  - US timezone indicators:
    - `et` = Eastern Time.
    - `ct` = Central Time.
    - `mt` = Mountain Time.
    - `pt` = Pacific Time.

<br>

#### Boolean column names:
  - Boolean columns should be prefixed with a present or past tense third-person singular verb, such as:
    - `is_` or `was_`.
    - `has_` or `had_`.
    - `does_` or `did_`.

<br>

#### Columns which represent numeric values with a known unit should be suffixed with that unit.
Some examples:
  - `price_usd`
  - `weight_oz`
  - `weight_kg`
  - `weight_grams`

<br>

#### Avoid using unnecessary table aliases, especially initialisms.
Suggested guidelines:
  - If the table name consists of 3 words or less don't alias it.
  - Use a subset of the words as the alias if it makes sense (e.g. if `partner_shipments_order_line_items` is the only line items table being referenced it could be reasonable to alias it as just `line_items`).

```sql
/* Good */
select
    customers.email
    , orders.invoice_number
from customers
inner join orders on customers.id = orders.customer_id

/* Bad */
select
    c.email
    , o.invoice_number
from customers as c
inner join orders as o on c.id = o.customer_id
```

<br>

## Formatting

An overarching pattern is:
  - If there's only one thing, put it on the same line as the opening keyword.
  - If there are multiple things, put each one on its own line (including the first one), indented one level more than the opening keyword.

<br>

#### Left align everything.
This is easier to keep consistent, and is also easier to write.

```sql
/* Good */
select email
from customers
where email like '%@domain.com'

/* Bad */
select email
  from customers
 where email like '%@domain.com'
```

<br>

#### Indents should generally be 4 spaces.
```sql
/* Good */
select
    id
    , email
from customers
where
    email like '%@domain.com'
    and plan_name != 'free'

/* Bad */
select
  id
  , email
from customers
where email like '%@domain.com'
  and plan_name != 'free'
```

<br>

#### Never end a line with an operator like `and`, `or`, `+`, `||`, etc.
If code containing such operators needs to be split across multiple lines, put the operators at the beginning of the subsequent lines.
  - You should be able to scan the left side of the query text to see the logic being used without having to read to the end of every line.
  - The operator is only there for/because of what follows it.  If nothing followed the operator it wouldn't be needed, so putting the operator on the same line as what follows it makes it clearer why it's there.

```sql
/* Bad */
select *
from customers
where
    email like '%@domain.com' and
    plan_name != 'free'

/* Good */
select *
from customers
where
    email like '%@domain.com'
    and plan_name != 'free'
```

<br>

#### Using leading commas.
If code containing commas needs to be split across multiple lines, put the commas at the beginning of the subsequent lines, followed by a space.
  - This makes it easier to spot missing commas.
  - Version control diffs will be cleaner when adding to the end of a list because you don't have to add a trailing comma to the preceding line.
  - The comma is only there for/because of what follows it.  If nothing followed the comma it wouldn't be needed, so putting the comma on the same line as what follows it makes it clearer why it's there.

```sql
/* Good */
with
    customers as (
        ...
    )
    , paying_customers as (
        ...
    )
select
    id
    , email
    , date_trunc('month', created_at) as signup_month
from paying_customers
where email in (
        'user-1@example.com'
        , 'user-2@example.com'
        , 'user-3@example.com'
    )

/* Bad */
with
    customers as (
        ...
    ),
    paying_customers as (
        ...
    )
select
    id,
    email,
    date_trunc('month', created_at) as signup_month
from paying_customers
where email in (
        'user-1@example.com',
        'user-2@example.com',
        'user-3@example.com'
    )
```

<br>

#### `select` clause:
  - If there is only one column expression, put it on the same line as `select`.
  - If there are multiple column expressions, put each one on its own line (including the first one), indented one level more than `select`.
  - If there is a `distinct` qualifier, put it on the same line as `select`.

```sql
/* Good */
select id

/* Good */
select
    id
    , email

/* Bad */
select id, email

/* Bad */
select id
    , email

/* Good */
select distinct country

/* Good */
select distinct
    state
    , country

/* Bad */
select distinct state, country
```

<br>

#### `from` clause:
  - Put the initial table being selected from on the same line as `from`.
  - If there are other tables being joined:
    - Put each `join` on its own line, at the same indentation level as `from`.
    - If there is only one join condition, put it on the same line as the `join`.
    - If there are multiple join conditions, end the `join` line with `on` and put each condition on its own line (including the first one), indented one level more than the `join`.

```sql
/* Good */
from customers

/* Good */
from customers
left join orders on customers.id = orders.customer_id

/* Bad */
from customers
    left join orders on customers.id = orders.customer_id

/* Bad */
from customers
left join orders
    on customers.id = orders.customer_id

/* Good */
from customers
left join orders on
    customers.id = orders.customer_id
    and customers.region_id = orders.region_id

/* Bad */
from customers
left join orders on customers.id = orders.customer_id
    and customers.region_id = orders.region_id

/* Bad */
from customers
left join orders
    on customers.id = orders.customer_id
    and customers.region_id = orders.region_id
```

<br>

#### `where` clause:
  - If there is only one condition, put it on the same line as `where`.
  - If there are multiple conditions, put each one on its own line (including the first one), indented one level more than `where`.

```sql
/* Good */
where email like '%@domain.com'

/* Good */
where
    email like '%@domain.com'
    and plan_name != 'free'

/* Bad */
where email like '%@domain.com' and plan_name != 'free'

/* Bad */
where email like '%@domain.com'
    and plan_name != 'free'
```

<br>

#### `group by` and `order by` clauses:
  - If grouping/ordering by column numbers, put all numbers on the same line as `group by`/`order by`.
  - If grouping/ordering by column names/aliases:
    - If there is only one column, put it on the same line as `group by`/`order by`.
    - If there are multiple columns, put each on its own line (including the first one), indented one level more than `group by`/`order by`.

```sql
/* Good */
group by 1, 2, 3

/* Bad */
group by
    1
    , 2
    , 3

/* Good */
order by plan_name

/* Good */
order by
    plan_name
    , signup_month

/* Bad */
order by plan_name, signup_month

/* Bad */
order by plan_name
    , signup_month
```

<br>

#### CTEs:
  - Start each CTE on its own line, indented one level more than `with` (including the first one, and even if there is only one).
  - Use a single blank line around CTEs to add visual separation.
  - Put any comments about the CTE within the CTE's parentheses, at the same indentation level as the `select`.

```sql
/* Good */
with
    paying_customers as (
        select ...
        from customers
    )

select ...
from paying_customers

/* Bad */
with paying_customers as (

    select ...
    from customers

)
select ...
from paying_customers

/* Good */
with
    paying_customers as (
        select ...
        from customers
    )

    , paying_customers_per_month as (
        /* CTE comments... */
        select ...
        from paying_customers
    )

select ...
from paying_customers_per_month

/* Bad */
with paying_customers as (

        select ...
        from customers

    )

    /* CTE comments... */
    , paying_customers_per_month as (

        select ...
        from paying_customers

      )

select ...
from paying_customers_per_month
```

<br>

#### `case` statements:
  - You can put a `case` statement all on one line if it only has a single `when` clause and doesn't cause the line's length to be too long.
  - For multi-line `case` statements:
    - `when`:
      - `when` clauses should start on their own line, indented one level more than the `case` statement.
      - If a `when` clause has multiple conditions, keep the first condition on the same line as `when` and put subsequent conditions on their own lines.
      - If a `when` clause has multiple lines, all its subsequent lines should be indented at least one level more than `when`.
    - `then`:
      - `then` clauses can go on the same line as a single-line `when` clause if it doesn't cause the line's length to be too long.
      - Otherwise, `then` clauses should go on their own line, indented one level more than its associated `when`.
      - If a `then` clause has multiple lines, all its subsequent lines should be indented at least one level more than `then`.
    - `else`:
      - `else` clauses should go on their own line, at the same indentation level as the `when` clauses.
      - If an `else` clause has multiple lines, all its subsequent lines should be indented at least one level more than `else`.
    - `end`:
      - `end` should go on its own line, at the same indentation level as `case`.
      - If the `case` starts after a leading comma and space, align `end` with `case` by adding two extra spaces before it.
    - If a multi-line `case` statement is within a function call, `case` and `end` should go on their own lines, indented one level more than the function call.
  - If using `case <expression>` syntax, keep the expression on the same line as `case`.

```sql
/* Good */
select
    case when customers.status_code = 1 then 'Active' else 'Inactive' end as customer_status

/* Bad */
select
    case when customers.status_code = 1 and customers.deleted_at is null then 'Active' else 'Inactive' end as customer_status


/* Good */
select
    case
        when customers.status_code = 1 then 'Active'
        else 'Inactive'
    end as customer_status
    , ...

/* Bad */
select
    case when customers.status_code = 1 then 'Active'
         else 'Inactive' end as customer_status
    , ...


/* Good */
select
    ...
    , case
        when customers.status_code = 1
            and customers.deleted_at is null
            then 'Active'
        else 'Inactive'
      end as customer_status

/* Bad */
select
    ...
    , case
        when customers.status_code = 1 and customers.deleted_at is null
        then 'Active'
        else 'Inactive'
    end as customer_status


/* Good */
select
    ...
    , sum(
        case
            when customers.status_code = 1
                and customers.deleted_at is null
                then customers.lifetime_value
            else 0
        end
      ) as active_customers_lifetime_value

/* Bad */
select
    ...
    , sum(case
          when customers.status_code = 1 and customers.deleted_at is null then customers.lifetime_value
          else 0
      end) as active_customers_lifetime_value
```

<br>

#### Window functions:
  - You can put a window function all on one line if it doesn't cause the line's length to be too long.
  - If breaking a window function into multiple lines:
    - Put each sub-clause within `over ()` on its own line, indented one level more than the window function:
      - `partition by`
      - `order by`
      - `rows between` / `range between`
    - Put the closing `over ()` parenthesis on its own line at the same indentation level as the window function.

```sql
/* Good */
select
    customer_id
    , invoice_number
    , row_number() over (partition by customer_id order by created_at) as order_rank
from orders

/* Good */
select
    customer_id
    , invoice_number
    , row_number() over (
        partition by customer_id
        order by created_at
      ) as order_rank
from orders

/* Bad */
select
    customer_id
    , invoice_number
    , row_number() over (partition by customer_id
                         order by created_at) as order_rank
from orders
```

<br>

#### `in` lists:
  - Break long lists of `in` values into multiple indented lines with one value per line.

```sql
/* Good */
where email in (
        'user-1@example.com'
        , 'user-2@example.com'
        , 'user-3@example.com'
    )

/* Bad */
where email in ('user-1@example.com', 'user-2@example.com', 'user-3@example.com')
```

<br>

#### Don't put extra spaces inside of parentheses.
```sql
/* Bad */
select *
from customers
where plan_name in ( 'monthly', 'yearly' )

/* Good */
select *
from customers
where plan_name in ('monthly', 'yearly')
```

<br>

## Credits

This style guide was inspired in part by:
  - [Fishtown Analytics' dbt coding conventions](https://github.com/fishtown-analytics/corp/blob/b5c6f55b9e7594e1a1e562edf2378b6dd78a1119/dbt_coding_conventions.md)
  - [Matt Mazur's SQL style guide](https://github.com/mattm/sql-style-guide/blob/3eaef3519ca5cc7f21feac6581b257638f9b1564/README.md)
  - [GitLab's SQL style guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)
