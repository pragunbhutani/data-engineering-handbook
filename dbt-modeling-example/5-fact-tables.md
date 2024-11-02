# Fact Tables

So far in our data modeling journey, we have:

1. Defined the source tables that are used to run our application.
2. Created staging tables that clean and rename columns from the source tables.
3. Created snapshots for some of our staging tables to track changes over time.
4. Used those snapshots to create dimension tables that represent the entities in our business.

Now, we will create fact tables that represent the numerical data or actions that represent the business processes of an organization.
Along with dimensions, facts are the other main component of a star schema.

In our e-commerce example, we will create the following fact tables:

1. **fct_orders**: Contains information about the orders placed by users.
2. **fct_order_items**: Contains information about the items in each order.

We will first create the `fct_order_items` table since it is a more granular version of `fct_orders`.
We will then aggregate this table to create the `fct_orders` table.

<Note>

You can find all the code for creating these fact tables in the `models/core/facts` directory
of the [example-dbt-project](https://github.com/pragunbhutani/example-dbt-project/tree/main/models/core/facts).

</Note>

## Model: fct_order_items

The `fct_order_items` table contains information about the items in each order.

### Specification

We would like our `fct_order_items` table to contain the following columns:

| Column Name                      | Type      | Description                                           |
| -------------------------------- | --------- | ----------------------------------------------------- |
| order_item_id                    | PK        | A unique identifier for each order item.              |
| order_id                         | FK        | Ref. `order_id` in the `stg_orders` table.            |
| item_id                          | FK        | Ref. `item_id` in the `stg_items` table.              |
| promotion_id                     | FK        | Ref. `promotion_id` in the `stg_promotions` table.    |
| dim_item_key                     | SCD Key   | Ref. `item_key` in the `dim_items_history` table.     |
| dim_promotion_key                | SCD Key   | Ref. `promotion_key` in the `dim_promotions` table.   |
| quantity                         | Measure   | The quantity of the item in the order.                |
| base_unit_selling_price_usd      | Measure   | The price of the item before applying any promotions. |
| effective_unit_selling_price_usd | Measure   | The price of the item after applying any promotions.  |
| landed_unit_cost_price_usd       | Measure   | The cost price of the item.                           |
| unit_margin_usd                  | Measure   | The margin on the item.                               |
| total_selling_price_usd          | Measure   | The total selling price of the item.                  |
| total_cost_price_usd             | Measure   | The total cost price of the item.                     |
| total_margin_usd                 | Measure   | The total margin on the item.                         |
| created_at                       | Dimension | Time when the order item was created.                 |
| updated_at                       | Dimension | Time when the order item was last updated.            |

You'll notice that not all these columns come from the same place. This is quite normal in a data warehouse.
A fact table is designed to bring together data from different sources to provide a complete picture of a business process and
doing so often requires going through multiple transformations.

Let us walk through the transformations required to create the `fct_order_items` table.

### Intermediates

Most of the information we need for the model can be found directly in one table or another.
However, the calculation of the `landed_unit_cost_price_usd` requires a bit more work.

We will create an intermediate model called `int_items_landed_cost_daily` to calculate this value.

Doing so allows us to abstract the complexity of the calculation and reuse it in other models if needed.

#### int_items_landed_cost_daily

If you go back to how the e-commerce platform works, you'll recall that merchants receive items from
vendors in batches daily. The price of an item in each batch is different and depends on the vendor it was bought from,
how large the batch was etc.

The landed cost price is therefore recalculated every time a new batch of items is received. It is calculated as the
weighted average of the items currently in stock and the new items received.

It looks like obtaining this number might take a fair amount of calculation, we will therefore relegate it to its own
intermediate model called `int_items_landed_cost_daily`.

The model should look something like this:

```sql
with

--
inventory as (
    select * from {{ ref('stg_inventory') }}
),

--
orders as (
    select * from {{ ref('stg_orders') }}
),

--
order_items as (
    select * from {{ ref('stg_order_items') }}
),

--
daily_sales as (
    select
        o.order_date as report_date,
        o.merchant_id,
        oi.item_id,
        sum(oi.quantity) as quantity_sold

    from
        orders as o
    join
        order_items as oi
            on o.order_id = oi.order_id

    group by
        1, 2, 3
),

--
daily_data as (
    select
        coalesce(i.acquisition_date, ds.report_date) as report_date,
        coalesce(i.merchant_id, ds.merchant_id) as merchant_id,
        coalesce(i.item_id, ds.item_id) as item_id,
        coalesce(i.quantity, 0) as quantity_acquired,
        coalesce(i.unit_cost_usd, 0) as unit_cost_usd,
        coalesce(ds.daily_quantity_sold, 0) AS quantity_sold

    from
        inventory as i
    full outer join
        daily_sales as ds
            on i.merchant_id = ds.merchant_id
            and i.item_id = ds.item_id
            and i.date_acquired = ds.report_date
),

--
daily_summary as (
    select
        *,

        quantity_acquired * unit_cost_usd as total_cost_usd,

        sum(quantity_acquired) over (
            partition by merchant_id, item_id
            order by report_date
        ) as cumulative_quantity_acquired,

        sum(total_cost_usd) over (
            partition by merchant_id, item_id
            order by report_date
        ) as cumulative_acquisition_cost_usd,

        cumulative_acquisition_cost_usd / cumulative_quantity_acquired as landed_unit_cost_usd

    from
        daily_data
)

--
select * from daily_summary

```

### Final Model

The final model for `fct_order_items` should look something like this:

```sql
with

--
order_items as (
    select * from {{ ref('stg_order_items') }}
),

--
orders as (
    select * from {{ ref('stg_orders') }}
),

--
items as (
    select * from {{ ref('dim_items_history') }}
),

--
promotions as (
    select * from {{ ref('dim_promotions_history') }}
),

--
item_landing_cost as (
    select * from {{ ref('int_items_landed_cost_daily') }}
),

--
collected_data as (
    select
        oi.order_item_id,
        oi.order_id,

        oi.item_id,
        oi.promotion_id,

        i.dim_item_key,
        p.dim_promotion_key,

        oi.quantity,
        coalesce(p.discount_percentage_as_float, 0) as discount_percentage_as_float,
        i.base_price_usd as base_unit_selling_price_usd,
        ilc.landed_unit_cost_usd as landed_unit_cost_price_usd,

        oi.created_at,
        oi.updated_at

    from
        order_items as oi
    join
        orders as o
            on oi.order_id = o.order_id
    join
        items as i
            on oi.item_id = i.item_id
            and oi.created_at between i.valid_from and i.valid_to
    join
        item_landing_cost as ilc
            on oi.item_id = ilc.item_id
            and oi.created_at = ilc.report_date
    left join
        promotions as p
            on oi.item_id = p.item_id
            and oi.created_at between p.valid_from and p.valid_to
),

--
final as (
    select
        *,

        base_unit_selling_price_usd * (1 - discount_percentage_as_float)
        as effective_unit_selling_price_usd,

        landed_unit_cost_price_usd - effective_unit_selling_price_usd
        as unit_margin_usd,

        quantity * effective_unit_selling_price_usd as total_selling_price_usd,
        quantity * landed_unit_cost_price_usd as total_cost_price_usd,
        quantity * unit_margin_usd as total_margin_usd

    from
        collected_data
)

--
select * from final
```

This creates the `fct_order_items` table that contains all the information we need about the items in each order.
The model is designed to be flexible and can be easily extended to include more information if needed.

It contains measures like `total_selling_price_usd`, `total_cost_price_usd`, and `total_margin_usd` that can be used to
analyze the performance of the business and calculate metrics like profit margins.

It also contains dimension keys that can be used to join with the dimension tables to get more
information about the entities involved at the time of the order.

The IDs of all dimensions are also included as they can be sometimes required for debugging or auditing purposes.

We can now use this table to create the `fct_orders` table that aggregates the data at the order level.

## Model: fct_orders

The model for `fct_orders` is quite simple. It aggregates the data from `fct_order_items` at the order level.

### Specification

We would like our `fct_orders` table to contain the following columns:

| Column Name             | Type      | Description                                               |
| ----------------------- | --------- | --------------------------------------------------------- |
| order_id                | PK        | A unique identifier for each order.                       |
| user_id                 | FK        | Ref. `user_id` in the `stg_users` table.                  |
| merchant_id             | FK        | Ref. `merchant_id` in the `stg_merchants` table.          |
| dim_user_key            | SCD Key   | Ref. `user_key` in the `dim_users_history` table.         |
| dim_merchant_key        | SCD Key   | Ref. `merchant_key` in the `dim_merchants_history` table. |
| order_placed_at         | Dimension | Date and time when the order was placed.                  |
| total_selling_price_usd | Measure   | The total selling price of the order.                     |
| total_cost_price_usd    | Measure   | The total cost price of the order.                        |
| total_margin_usd        | Measure   | The total margin on the order.                            |
| created_at              | Dimension | Time when the order was created.                          |
| updated_at              | Dimension | Time when the order was last updated.                     |

### Final Model

The final model for `fct_orders` should look something like this:

```sql
with

--
order_items as (
    select * from {{ ref('fct_order_items') }}
),

--
orders as (
    select * from {{ ref('stg_orders') }}
),

--
users as (
    select * from {{ ref('dim_users_history') }}
),

--
merchants as (
    select * from {{ ref('dim_merchants_history') }}
),

--
order_items_aggregated as (
    select
        order_id,
        sum(total_selling_price_usd) as total_selling_price_usd,
        sum(total_cost_price_usd) as total_cost_price_usd,
        sum(total_margin_usd) as total_margin_usd

    from
        order_items

    group by
        1
),

--
final as (
    select
        o.order_id,
        o.user_id,
        o.merchant_id,
        u.dim_user_key,
        m.dim_merchant_key,
        o.order_placed_at,
        oi.total_selling_price_usd,
        oi.total_cost_price_usd,
        oi.total_margin_usd,
        o.created_at,
        o.updated_at

    from
        order_items_aggregated as oi
    join
        orders as o
            on o.order_id = oi.order_id
    join
        users as u
            on o.user_id = u.user_id
            and o.created_at between u.valid_from and u.valid_to
    join
        merchants as m
            on o.merchant_id = m.merchant_id
            and o.created_at between m.valid_from and m.valid_to
)

--
select * from final
```

## Next Steps

We have now created the fact tables for our e-commerce data warehouse.
These tables contain the numerical data that represents the business processes of an organization.

The `fct_order_items` table contains information about the items in each order and the
`fct_orders` table aggregates this data at the order level.

With our facts and dimensions in place, we have a complete star schema data model that can be
used for analytics and reporting purposes.

In the next and final section, we will recap everything we have done so far and look at an
example of creating a mart table that aggregates data into Gross Merchandise Value (GMV) and margin metrics.
