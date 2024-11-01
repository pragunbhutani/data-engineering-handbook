# Dimension Tables

So far in our data modeling journey, we have:

1. Defined the schema of the source tables.
2. Created a staging layer to clean and rename the columns.
3. Created snapshots of the staging tables to track changes over time.

We shall now begin creating our dimension tables. **Dimensions** are descriptive attributes that provide context to facts.
They are typically used for filtering and grouping data. Examples of dimensions include time, location, and product.

When you track the changes to a dimension over time, it is known as a **Slowly Changing Dimension**.

<Note>

Our dimension tables follow the following naming convention:

- `dim_<entity_name>` for simple dimensions.
- `dim_<entity_name>_history` for slowly changing dimensions.

</Note>

<Note>

You can find all the code for creating these dimension tables in the `models/core/dimensions/` directory
of the [example-dbt-project](https://github.com/pragunbhutani/example-dbt-project/tree/main/models/core/dimensions).

</Note>

## Creating Dimension Tables

### dim_users_history

#### Specification

We would like our `dim_users_history` table to have the following columns.

Note that our `user_id` is no longer a primary key in this table.
This is because since we can now have multiple rows for each user, each row representing a different version of the user's history,
the `user_id` is no longer unique.

Instead, we have a `dbt_scd_id` column that acts as the primary key.

We have also added a column called `is_current` that indicates whether a row is the current version of the user.

| Column Name   | Type      | Description                                    |
| ------------- | --------- | ---------------------------------------------- |
| dim_users_key | PK        | Primary Key                                    |
| user_id       | FK        | Foreign Key referencing user_id in users table |
| user_name     | varchar   | Name of the user                               |
| user_email    | varchar   | Email address of the user                      |
| latitude      | float     | Latitude of the user's location                |
| longitude     | float     | Longitude of the user's location               |
| created_at    | timestamp | Time when the user was created                 |
| updated_at    | timestamp | Time when the user was last updated            |
| valid_from    | timestamp | Time when the record became valid              |
| valid_to      | timestamp | Time when the record ceased to be valid        |
| is_current    | boolean   | Indicates whether the record is current        |

#### Model

The following dbt model creates a slowly changing dimension table for the `users` table.

```sql
with

--
source as (
    select * from {{ ref('snapshot_stg_users') }}
),

--
renamed as (
    select
        dbt_scd_id as dim_users_key,
        user_id,

        user_name,
        user_email,
        latitude,
        longitude,

        created_at,
        updated_at,

        dbt_valid_from as valid_from,
        dbt_valid_to as valid_to,

        (valid_to is null) as is_current

    from
        source
)

--
select * from renamed
```

### dim_merchants_history

#### Specification

We would like our `dim_merchants_history` table to have the following columns:

| Column Name       | Type      | Description                                            |
| ----------------- | --------- | ------------------------------------------------------ |
| dim_merchants_key | PK        | Primary Key                                            |
| merchant_id       | FK        | Foreign Key referencing merchant_id in merchants table |
| merchant_name     | varchar   | Name of the merchant                                   |
| latitude          | float     | Latitude of the merchant's location                    |
| longitude         | float     | Longitude of the merchant's location                   |
| created_at        | timestamp | Time when the merchant was created                     |
| updated_at        | timestamp | Time when the merchant was last updated                |
| valid_from        | timestamp | Time when the record became valid                      |
| valid_to          | timestamp | Time when the record ceased to be valid                |
| is_current        | boolean   | Indicates whether the record is current                |

#### Model

The following dbt model creates a slowly changing dimension table for the `merchants` table.

```sql
with

--
source as (
    select * from {{ ref('snapshot_stg_merchants') }}
),

--
renamed as (
    select
        dbt_scd_id as dim_merchants_key,
        merchant_id,

        merchant_name,
        latitude,
        longitude,

        created_at,
        updated_at,

        dbt_valid_from as valid_from,
        dbt_valid_to as valid_to,

        (valid_to is null) as is_current

    from
        source
)

--
select * from renamed
```

### dim_items_history

#### Specification

We would like our `dim_items_history` table to have the following columns:

| Column Name    | Type      | Description                                    |
| -------------- | --------- | ---------------------------------------------- |
| dim_items_key  | PK        | Primary Key                                    |
| item_id        | FK        | Foreign Key referencing item_id in items table |
| item_name      | varchar   | Name of the item                               |
| item_category  | varchar   | Category of the item                           |
| base_price_usd | float     | Base price of the item in USD                  |
| created_at     | timestamp | Time when the item was created                 |
| updated_at     | timestamp | Time when the item was last updated            |
| valid_from     | timestamp | Time when the record became valid              |
| valid_to       | timestamp | Time when the record ceased to be valid        |
| is_current     | boolean   | Indicates whether the record is current        |

#### Model

The following dbt model creates a slowly changing dimension table for the `items` table.

```sql
with

--
source as (
    select * from {{ ref('snapshot_stg_items') }}
),

--
renamed as (
    select
        dbt_scd_id as dim_items_key,
        item_id,

        item_name,
        item_category,
        base_price_usd,

        created_at,
        updated_at,

        dbt_valid_from as valid_from,
        dbt_valid_to as valid_to,

        (valid_to is null) as is_current

    from
        source
)

--
select * from renamed
```

### dim_promotions_history

#### Specification

We would like our `dim_promotions_history` table to have the following columns:

| Column Name                  | Type      | Description                                              |
| ---------------------------- | --------- | -------------------------------------------------------- |
| dim_promotions_key           | PK        | Primary Key                                              |
| promotion_id                 | FK        | Foreign Key referencing promotion_id in promotions table |
| item_id                      | FK        | Foreign Key referencing item_id in items table           |
| discount_percentage_as_float | float     | Percentage discount applied to the item                  |
| created_at                   | timestamp | Time when the promotion was created                      |
| updated_at                   | timestamp | Time when the promotion was last updated                 |
| valid_from                   | timestamp | Time when the record became valid                        |
| valid_to                     | timestamp | Time when the record ceased to be valid                  |
| is_current                   | boolean   | Indicates whether the record is current                  |

The following dbt model creates a slowly changing dimension table for the `promotions` table.

```sql
with

--
source as (
    select * from {{ ref('snapshot_stg_promotions') }}
),

--
renamed as (
    select
        dbt_scd_id as dim_promotions_key,
        promotion_id,

        item_id,
        discount_percentage,

        created_at,
        updated_at,

        dbt_valid_from as valid_from,
        dbt_valid_to as valid_to,

        (valid_to is null) as is_current

    from
        source
)

--
select * from renamed
```

## Next Steps

We have now created our dimension tables. In the next section, we shall create our fact tables.
