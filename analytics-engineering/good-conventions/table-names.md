# Table Names

## Basic conventions

Model names should be made of the following components:

- **Prefix**: Type of model
- **Name**: Descriptive name of the model
- **Suffix**: Optional additional context

**Table names should be:**

- Lowercase.
- Written in snake_case, with single underscores.
- Plural. For example, dim_customers ✅ vs dim_customer ❌.
- Descriptive enough to understand the content of the model without having to look at the model itself.
- Concisely named, but not so short that the name is ambiguous.

Following are a few examples of model names that follow the above conventions:

> **Examples**:
>
> - stg_salesforce_opportunities
> - fct_chargebee_subscriptions
> - fct_customer_revenue_monthly
> - dim_customers_history

## Prefixes

The prefix should be a short abbreviation that describes the type and role of the model.
These roles correspond to the different stages of the ELT process and will usually have a corresponding schema in the database.

Refer to the [Warehouse Layers](/good-conventions/warehouse-layers) section for more information on the different schema types.

Following is a list of common warehouse layers and their corresponding prefixes:

- `raw_`: Raw data from source systems.
- `stg_`: Staging area for data transformation.
- `int_`: Intermediate models that are created during the transformation process.
- `snap_`: Snapshot models that store historical data.
- `fct_`: Fact tables.
- `dim_`: Dimension tables.
- `mart_`: Mart tables.

### Raw Data

**Prefix**: `raw_`

Raw table names should be the same as the source table names or as close as possible.

> **Examples**
>
> The replica of the table `users` from `main_db` should be called `raw_main_db_users`.

### Staging

**Prefix**: `stg_`

Staging models should also contain the source system that the data is coming from.

> **Examples**
>
> `stg_salesforce_contracts` or `stg_zuora_subscriptions`.

#### Staging Partials

**Prefix**: `stg_part_`

Partial staging tables may be used in the cases where the source data exists in multiple tables or files or is otherwise split into multiple parts.

> **Examples**
>
> `stg_part_accounts_deleted` and `stg_part_accounts_active` which are then combined into `stg_accounts`.

### Intermediate

**Prefix**: `int_`

Intermediate tables that are created during the transformation process.
The name of a table should describe the data that is being transformed along with the nature of the transformation.

> **Examples**
>
> A denormalised table that combines Salesforce accounts with their parent accounts may be called `int_salesforce_accounts_with_parent`.

### Snapshot

**Prefix**: `snap_`

Snapshot models should be suffixed with the name of the model being snapshotted.

> **Examples**
>
> A table that snapshots `int_salesforce_accounts_with_parent` should be called `snap_int_salesforce_accounts_with_parents`.

### Facts

**Prefix**: `fct_`

Fact tables should be named after the most granular entity that the fact table is about.

#### Aggregated Facts

For aggregated fact tables, the name should include all the dimensions that are being factored, as well as the frequency of the data.

This will help to differentiate between different fact tables that may exist for the same entity - for example, monthly revenue vs daily revenue.

> **Examples**
>
> - A fact table that contains all messages sent between users could be called `fct_user_messages`.
> - A fact table that contains monthly revenue data should be called `fct_accounts_revenue_monthly`.

### Dimensions

**Prefix**: `dim_`

Dimension tables should be named after the entity that the dimension is about. By default, it is assumed that the dimension table only contains the most recent data.

#### Slowly Changing Dimensions

For slowly changing dimensions, the table should be suffixed with `_history` to indicate that it contains historical data.

> **Examples**
>
> - A dimension table that contains information about customers should be called `dim_customers`.
> - A slowly changing dimension table that contains historical data about customers should be called `dim_customers_history`.

### Marts

**Prefix**: `mart_`

Mart tables or "exposures" are tables that are used for reporting and analysis. They are usually a combination of facts and dimensions.
They areused to expose data to end-users, are denormalised for ease of use and usually serve a specific business function or use case.

These tables should be named after the business function or use case that they serve.

> **Examples**
>
> - A mart table that contains customer retention cohort data could be called `mart_customer_retention_cohorts`.
> - A mart table used to create customer segments for online ads could be called `mart_online_ads_customer_segments`.
