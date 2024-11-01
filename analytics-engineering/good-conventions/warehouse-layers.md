# Warehouse Layers

Good warehouse layers are the foundation of a well-designed data warehouse.
They provide a clear separation of concerns between the different stages of your ETL process and
make it easier to manage and maintain your data warehouse.

Separating your data into layers also allows you to control who gets to see what data and improve data discoverability.

As a general principle, you should at least have the following layers in your data warehouse.

- [Source](#source)
- [Intermediate](#intermediate)
- [Core](#core)
- [Marts](#marts)

You should separate your staging, intermediate and consumption layer data tables into separate schemas.

## Source

Source layers can be of two types - Raw Data (with no transformation) or Staging (with minimal data cleaning and homogenisation).

### Raw Data

Contains replicas of data as it is at source, without any transformation or changes whatsoever.
Table and column names should match those in the original source without any modifications.

Create a separate namespace for each data source. Each production database or cluster should be considered a separate source.

This layer should only be visible to builders of ETL pipelines and not to end users.

### Staging

Contains views on top of your raw data layer that contain the following transformations:

- Combining partial data together into a single table. e.g. Combining active and deleted records.
- Rename columns to follow a consistent naming scheme across all your tables.
- Type cast columns for consistency or ease of use. e.g. converting all timestamps to UTC,
  all boolean columns to TRUE/FALSE instead of 1/0 etc.

This layer should only be visible to builders of ETL pipelines and not to end users.

## Intermediate

The main purpose of this layer is to provide a space where you can store the intermediate results of your ETL process without
crowding the user facing core and mart layers.

These tables often don't map directly to business concepts but are used as stepping stones to
create the core and mart layers and can be useful for debugging and troubleshooting larger transformations.

This layer should only be visible to builders of ETL pipelines and not to end users.

## Core

The core layer stores the facts and dimensions that are the foundation of your data warehouse.
These are the tables that map directly to business concepts and are the tables that you will be querying most often.

This layer should be visible to all users and should be the single source of truth for your data.

## Marts

Marts are the denormalised views on top of your core layer that are optimised for specific use cases.
They are often used to speed up queries for specific use cases and can be used to create reports and dashboards.
