# Data Modeling with dbt

In this article, we'll walk through the process of using dbt (Data Build Tool) to create a
Kimball-style data warehouse for an e-commerce store with a mobile application.

Our goal is to model data from production tables, transform it into staging tables, and finally
create facts and dimensions, including slowly changing dimensions.

We will also demonstrate how to use intermediate models to store the results of different stages of the
ETL (Extract, Transform, Load) process.

Finally, we'll create a mart layer that aggregates data into Gross Merchandise Value (GMV) and margin metrics.

> [!NOTE]
> You can find all the code for this example in the
> [example-dbt-project](https://github.com/pragunbhutani/example-dbt-project) repository on GitHub.

## Table of Contents

The chapters are to be read in order as they build on each other.

1. [Source Data](/dbt-modeling-example/1-source-data.md)
2. [Staging Layer](/dbt-modeling-example/2-staging-layer.md)
3. [Snapshots](/dbt-modeling-example/3-snapshots.md)
4. [Dimension Tables](/dbt-modeling-example/4-dimension-tables.md)
5. [Fact Tables](/dbt-modeling-example/5-fact-tables.md)
6. [Conclusion](/dbt-modeling-example/6-conclusion.md)
