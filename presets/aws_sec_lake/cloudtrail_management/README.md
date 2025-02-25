
# AWS Security Lake CloudTrail Management Event presets

This preset directory contains AWS Security Lake data sources. Current supported data sources:
- CloudTrail Management

## CloudTrail Management Event

Assumed data format: parquet

Silver tables:
- aws_sec_lake_cloud_trail_mgmt_account_change
- aws_sec_lake_cloud_trail_mgmt_authentication
- aws_sec_lake_cloud_trail_mgmt_api_activity

Gold tables:
- account_change
- authentication
- api_activity
