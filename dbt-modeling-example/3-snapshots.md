# Snapshots

So far in our data modeling journey, we've looked at the schema of the source tables and created
a staging layer to clean and rename the columns. We shall now begin working on creating the core
layer of a data warehouse that consists of facts and dimensions.

<Note>

Facts and dimensions are the building blocks of a Kimball-style data warehouse.

- **Facts** are measurements, metrics or actions that represent business processes. They are typically
  numeric values that can be aggregated. Examples of facts include sales revenue, quantity sold, and
  the number of orders placed.
- **Dimensions** provide context to facts. They are descriptive attributes that help to understand the
  facts. Examples of dimensions include time, location, and product.

This warehouse architecture is also known as a star schema. You can find detailed information about
the start schema on the [Kimball Group website](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/).

</Note>

We will first create the dimension tables and then move on to the fact tables. In order to create our dimension tables,
we need to keep track of changes to our data over time. This is where the `snapshot` functionality in dbt comes in handy.

## What is a Snapshot?

The snapshot functionality in dbt compares the current state of a table with its previous state and creates a new row
every time it detects a change. This allows us to track changes to our data over time and create a history of our data.
dbt offers two strategies to compare the current state of a table with its previous state: `timestamp` and `check_cols`.

- **Timestamp**: This strategy compares the current state of a table with its previous state based on a
  timestamp column - usually an `updated_at` column.
  If the timestamp column is updated, dbt creates a new row in the snapshot table.
- **Check_cols**: This strategy compares the current state of a table with its previous state based on the values
  of specific columns. If any of the columns specified in the `check_cols` configuration are updated, dbt creates a new row
  in the snapshot table.

<Note>

dbt recommends the `timestamp` strategy for most use cases. The `check_cols` strategy can be used when you don't have a
timestamp column in your table, however it is less performant than the `timestamp` strategy.

</Note>

### Example

The follow example was taken from [dbt documentation](https://docs.getdbt.com/docs/building-a-dbt-project/building-models/snapshots).

Imagine you have an `orders` table with the following columns:

| id  | status  | updated_at |
| --- | ------- | ---------- |
| 1   | pending | 2019-01-01 |
| 2   | pending | 2019-01-01 |

Now, imagine that the order goes from "pending" to "shipped". That same record will now look like:

| id  | status  | updated_at |
| --- | ------- | ---------- |
| 1   | shipped | 2019-01-02 |
| 2   | pending | 2019-01-01 |

This order is now in the "shipped" state, but we've lost the information about when the order was
last in the "pending" state.
This makes it difficult (or impossible) to analyze how long it took for an order to ship.
dbt can "snapshot" these changes to help you understand how values in a row change over time.

Here's an example of a snapshot table for the previous example:

| id  | status  | updated_at | valid_from | valid_to   |
| --- | ------- | ---------- | ---------- | ---------- |
| 1   | pending | 2019-01-01 | 2019-01-01 | 2019-01-02 |
| 1   | shipped | 2019-01-02 | 2019-01-02 | `null`     |
| 2   | pending | 2019-01-01 | 2019-01-01 | `null`     |

## Creating Snapshots

Let us now create snapshots of our staging tables. Snapshot tables are named with the prefix `snap_`
followed by the name of the model being snapshotted.
For example, the snapshot table for the `stg_users` table would be named `snap_stg_users`.

<Note>

You can find all the code for creating these snapshots in the `snapshots/` directory
of the [example-dbt-project](https://github.com/pragunbhutani/example-dbt-project/tree/main/snapshots/).

</Note>

### Users

```sql
{% snapshot snap_stg_users %}

{{
    config(
        target_schema='intermediate',
        unique_key='user_id',
        strategy='timestamp',
        updated_at='updated_at'
    )
}}

select * from {{ ref('stg_users') }}

{% endsnapshot %}
```

### Merchants

```sql
{% snapshot snap_stg_merchants %}

{{
    config(
        target_schema='intermediate',
        unique_key='merchant_id',
        strategy='timestamp',
        updated_at='updated_at'
    )
}}

select * from {{ ref('stg_merchants') }}

{% endsnapshot %}
```

### Items

```sql
{% snapshot snap_stg_items %}

{{
    config(
        target_schema='intermediate',
        unique_key='item_id',
        strategy='timestamp',
        updated_at='updated_at'
    )
}}

select * from {{ ref('stg_items') }}

{% endsnapshot %}
```

### Promotions

```sql
{% snapshot snap_stg_promotions %}

{{
    config(
        target_schema='intermediate',
        unique_key='promotion_id',
        strategy='timestamp',
        updated_at='updated_at'
    )
}}

select * from {{ ref('stg_promotions') }}

{% endsnapshot %}
```

## Next Steps

In this section, we created snapshot tables for our staging tables. These snapshot tables will help us track changes to our data
and serve as the basis for our dimension tables.

In the next section, we will use these snapshots to create the dimension tables for our warehouse.
