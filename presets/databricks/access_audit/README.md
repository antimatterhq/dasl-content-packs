# Databricks Audit Table Preset

This preset contains instructions for processing data found in the Databricks `system.access.audit` table within Unity Catalog. The audit table contains information from a wide variety of sources and so auditing data will flow into a number of gold OCSF tables.


## Bronze

A Databricks account's existing `system.access.audit` table is used for bronze data. No ingestion is needed for this datasource.

## Silver

The following silver tables are created by this preset:

- databricks_access_audit

## Gold

The following gold OCSF tables are targetted by this preset:

- account_change
- api_activity
- authentication
- group_management

## Further Information

- [Audit log reference](https://docs.databricks.com/aws/en/admin/account-settings/audit-logs)
- [Audit log system table reference](https://docs.databricks.com/aws/en/admin/system-tables/audit-logs)