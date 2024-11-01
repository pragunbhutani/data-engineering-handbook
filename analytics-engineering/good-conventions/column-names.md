# Column Names

Column names should be descriptive and follow a consistent naming convention.

## General Principles

**Optimise for clarity, not for brevity**.
Long names can be mentioned once and then aliased in queries but the confusion caused by unclear names can't be fixed as easily.

Some other general principles to follow:

- Column names should be lower snake cased.
- Use consistent and descriptive identifiers and names.
- Limit use of abbreviations that are related to domain knowledge.
- Avoid using reserved words as column names i.e. a date column should not be called date, use calendar_date instead.
- Names must begin with a letter and may not end in an underscore.

## Data Type Principles

### Boolean Columns

- Boolean columns should be stored as true and false rather than 1 and 0, etc
- Boolean columns should be prefixed with a present or past tense third-person singular verb, such as:
  - is* or was*.
  - has* or had*.
  - does* or did*.

### Date and Time Columns

- Date columns should be suffixed with `_date`, `_month`, `_year` depending on the level of granularity.
  - e.g.
    - `created_date`
    - `updated_month`
- Datetime columns should be suffixed with `_at`.
  - e.g.
    - `created_at`
    - `updated_at`
- Use UTC time zone by default. If you need to use another timezone, suffix the column name with the timezone identifier.
  - e.g.
    - `created_at` refers to the time in UTC.
    - `created_at_eu_fr` refers to the time in Europe/Paris timezone.

### Foreign Key Columns

- Foreign key columns should be named after the referenced table and suffixed with `_id`.
  - e.g.
    - `user_id`
    - `product_id`
- If the foreign key is part of a composite key, the column name should be prefixed with the table name.
  - e.g.
    - `order_user_id`
    - `order_product_id`
- If the foreign key is part of a self-referencing relationship, the column name should be suffixed with `_id` and prefixed with the relationship name.
  - e.g.
    - `manager_id`
    - `parent_id`

### Numeric Columns

- Numeric columns should be named after the quantity they represent.
  - e.g.
    - `quantity`
    - `price`
- Mention the unit of measurement in the column name if it is not obvious.
  - e.g.
    - `length_cm`
    - `weight_kg`

### ETL metadata

Columns storing ETL related metadata are prefixed with double underscores, e.g. `__dbt_inserted_at`

## Aggregate Columns

Aggregate columns should take the form:

`aggregation`\_`object`\_`context`\_`units`

**Aggregation**:

Should be:

- `count` when measuring the quantity of discrete objects e.g. conversations
- `sum` when measuring the quantity of continuous quantities e.g. ARR
- Otherwise the aggregation function should be stated explicitly:
  - e.g.
    - avg
    - median
    - std_dev
    - ...

**Object**

Refers to the underlying object being aggregated (e.g. apps, inbox conversations, outbound messages).

More abstract objects are allowed (e.g. paid trials, net new arr) but these should be agreed, broadly understood, and documented.

**Context**

Should provide information on the filtering applied to the object being aggregated.
This can be further split into qualifiers and actors: context = qualifier_by_actor

**Units**

Should be stated where applicable. In cases where we only use one unit (e.g. dollars for currency columns) they can be omitted.
However, if there is any ambiguity they should be included. Explicit is better than implicit.

**Examples**:

- A column containing the number of inbound support tickets should be called `count_support_tickets_inbound`
- A column containing the number of inbound support tickets from email should be called `count_support_tickets_inbound_from_email`
- A column containing the number of support tickets which had a teammate reply before they were closed should be called `count_support_tickets_resolved_by_teammate`
- A column containing the time in hours a teammate was active in the product should be called `sum_teammate_time_active_in_hours`
- A column containing the median ARR per app should be called `median_arr_per_app`
- A column containing the average cost per request in US dollars should be called `avg_cost_per_request_in_usd`
