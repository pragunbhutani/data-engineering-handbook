# Staging Layer

In the previous section, we listed the raw tables that we have in our database.
Every data modeling journey begins with similar raw tables.

These tables contain the data that is needed for your application to function.
Such tables often contain a lot of irregularities and inconsistencies in the way data is stored or
represented.
This irregularities are a result of the way data is collected and stored in the source systems.

This is where the staging layer comes in. The staging layer is used to clean and rename our columns so that
they are consistent and easy to work with.
This layer doesn't contain any transformations or business logic, it's just a copy of the raw tables
with some cleaning and renaming.

Let us work through the raw tables in our database and create a staging layer for each of them.

We prefix our table names with `stg_` to indicate that they are staging tables.

<Note>

You can find all the code for creating these staging tables in the `models/staging/product` directory
of the [example-dbt-project](https://github.com/pragunbhutani/example-dbt-project/tree/main/models/staging/product).

</Note>

## Create Staging Tables

### stg_users

The columns of our `users` table need the following changes:

- `user_id`
- `name` - Should be renamed to `user_name`
- `email_address` - Should be renamed to `user_email`
- `lat` - Should be renamed to `latitude`
- `long` - Should be renamed to `longitude`
- `createdAt` - Should be renamed to `created_at`
- `updatedAt` - Should be renamed to `updated_at`

The following dbt model creates a staging table for the `users` table.

```sql
with

--
source as (
    select * from {{ source('source', 'users') }}
),

--
renamed as (
    select
        user_id,
        name as user_name,
        email_address as user_email,
        lat as latitude,
        long as longitude,
        createdAt as created_at,
        updatedAt as updated_at
    from
        source
)

--
select * from renamed

```

### stg_merchants

The columns of our `merchants` table need the following changes:

- `merchant_id`
- `merchant_name`
- `geo_loc` - This is a custom datatype that stores "latitude/longitude" as a single value. We should split this into two columns `latitude` and `longitude`
- `createdAt` - Should be renamed to `created_at`
- `updatedAt` - Should be renamed to `updated_at`

The following dbt model creates a staging table for the `merchants` table.

```sql
with

--
source as (
    select * from {{ source('source', 'merchants') }}
),

--
renamed as (
    select
        merchant_id,
        name AS merchant_name,
        split(geo_loc, '/')[0] as latitude,
        split(geo_loc, '/')[1] as longitude,
        createdAt as created_at,
        updatedAt as updated_at

    from
        source
)

--
select * from renamed
```

### stg_items

The columns of our `items` table need the following changes:

- `item_id`
- `merchant_id`
- `name` - Should be renamed to `item_name` because explicit is better than implicit
- `category` - Should be renamed to `item_category` because explicit is better than implicit
- `price` - Should be renamed to `base_price_usd` so that it's clear that the price is in USD
- `created_at`
- `updated_at`

The following dbt model creates a staging table for the `items` table.

```sql
with

--
source as (
    select * from {{ source('source', 'items') }}
),

--
renamed as (
    select
        item_id,
        name as item_name,
        category as item_category,
        price as base_price_usd,
        created_at,
        updated_at

    from
        source
)

--
select * from renamed
```

### stg_promotions

The columns of our `promotions` table need the following changes:

- `promotion_id`
- `product_id` - The use of the word 'product' is because of legacy reasons. We should rename this to `item_id` to match the `items` table
- `discount` - By default we store the discount percentage as an integer e.g. If we offer 20% discount, this value is 20.
  We will convert this to a float by dividing by 100 to make it easier to work with. We will also rename it to `discount_percentage_as_float`
- `created_at`
- `updated_at`

The following dbt model creates a staging table for the `promotions` table.

```sql
with

--
source as (
    select * from {{ source('source', 'promotions') }}
),

--
renamed as (
    select
        promotion_id,
        product_id AS item_id,
        discount / 100 AS discount_percentage_as_float,
        created_at,
        updated_at

    FROM
        source
)

--
select * from renamed
```

### stg_inventory

The columns of our `inventory` table need the following changes:

- `inventory_id`
- `merchant_id`
- `item_id`
- `qty` - Should be renamed to `quantity` for clarity
- `date_acquired` - Should be renamed to `acquisition_date` because we keep the units as suffixes
- `cost` - Should be renamed to `unit_cost_usd` so that it's clear that the price is in USD and we're talking about the cost of a single unit
- `created_at`
- `updated_at`

We'll also add a column `total_cost_usd` which is the product of `quantity` and `unit_cost_usd` since this is a common calculation that we'll need to do.

The following dbt model creates a staging table for the `inventory` table.

```sql
with

--
source as (
    select * from {{ source('source', 'inventory') }}
),

--
renamed as (
    from
        inventory_id,
        merchant_id,
        item_id,
        qty as quantity,
        date_acquired as acquisition_date,
        cost as unit_cost_usd,
        qty * cost as total_cost_usd,
        created_at,
        updated_at

    from
        source
)

--
select * from renamed
```

### stg_orders

The columns of our `orders` table need the following changes:

- `order_id`
- `customer_id` - Should be renamed to `user_id` to match the `users` table
- `store_id` - Should be renamed to `merchant_id` to match the `merchants` table
- `date` - Should be renamed to `order_placed_at` because it's a timestamp and not a date, and we want to clarify that it's the time when the order was placed
- `created_at`
- `updated_at`

The following dbt model creates a staging table for the `orders` table.

```sql
with

--
source as (
    select * from {{ source('source', 'orders') }}
)

--
renamed as (
    select
        order_id,
        customer_id as user_id,
        store_id as merchant_id,
        date as order_placed_at,
        created_at,
        updated_at

    from
        source
)

--
select * from renamed
```

### stg_order_items

The columns of our `order_items` table need the following changes:

- `order_item_id`
- `order_id`
- `product_id` - Should be renamed to `item_id` to match the `items` table
- `qty` - Should be renamed to `quantity` for clarity
- `created_at`
- `updated_at`

The following dbt model creates a staging table for the `order_items` table.

```sql
with

--
source as (
    select * from {{ source('source', 'order_items') }}
),

--
renamed as (
    select
        order_item_id,
        order_id,
        product_id as item_id,
        qty as quantity,
        created_at,
        updated_at

    from
        source
)

--
select * from renamed
```

## Next Steps

In this section, we have created staging tables for each of the raw tables in our database.
These staging tables are used to clean and rename our columns so that they are consistent and easy to work with.

The core of a data warehouse is made up of tables known as facts and dimensions.
Facts represent actions or events, while dimensions represent the context in which these actions or events occur.

In the next section, we will use the `snapshot` functionality in dbt to create snapshots of our staging tables.
These snapshots will allow us to track changes to our data over time and create a history of our data.

Such a table that contains historical records of your data is called a slowly changing dimension (SCD) table.

Once we have our SCD tables, we will create our fact tables.
