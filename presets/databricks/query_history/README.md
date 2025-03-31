# Databricks Query History Preset

This preset contains instructions for processing data found in the Databricks `system.query.history` table within Unity Catalog. All queries executed in a Databricks account are recorded in this table providing deep insight into data interactions.


## Bronze

A Databricks account's existing `system.query.history` table is used for bronze data. No ingestion is needed for this datasource.

## Silver

The following silver tables are created by this preset:

- databricks_query_history

## Gold

The following gold OCSF tables are targetted by this preset:

- api_activity

## Further Information

- [Query history system table reference ](https://docs.databricks.com/aws/en/admin/system-tables/query-history)
