
# AWS Security Lake Security Hub presets

This preset directory contains AWS Security Lake data sources. Current supported data sources:
- Security Hub

## Security Hub

Assumed data format: parquet

Silver tables:
- aws_sec_lake_security_hub_findings_vulnerability_finding
- aws_sec_lake_security_hub_findings_compliance_finding
- aws_sec_lake_security_hub_findings_detection_finding

Gold tables:
- vulnerability_finding
- compliance_finding
- detection_finding
