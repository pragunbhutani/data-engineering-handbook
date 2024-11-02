# ETL, ELT and Reverse ETL

I've added a dedicated page for these because these terms get mixed up a lot,
thanks in part to marketing jargon and I want to clarify them.

## ETL / ELT

### ETL

ETL stands for Extract, Transform, Load.

ETL is a process where data is extracted from a source, transformed and then loaded into a target - typically a data warehouse.
This meant that you would transform the data in transit from source to target, and often not store the raw data.

This was the traditional way of doing things when storage was expensive and processing power was limited.

### ELT

ELT stands for Extract, Load, Transform.

ELT is a process where data is extracted from a source, loaded into a target and then transformed.
This means that you first replicate the data as-is into the target and then transform it there itself.

Both raw and transformed data are stored in the target.

This is the most common way of doing things now that storage is cheap and processing power is abundant.

## Reverse ETL

Reverse ETL is a process where data is extracted from a target, transformed and then loaded into a source.
"Source" in this case commonly refers to business applications like CRMs, marketing automation tools, etc.

An example of a reverse ETL process would be to calculate user segments in a data warehouse and then
load them into a CRM to trigger marketing campaigns.
