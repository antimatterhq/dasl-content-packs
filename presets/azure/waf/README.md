# Azure presets

This preset directory contains Azure data sources. Current supported data sources:
- http request

## Http Request

Assumed data format: json

Silver tables:
- azure_waf_application_gateway_access_log
- azure_waf_application_gateway_firewall_log
- azure_waf_az_fw_application_rule
- azure_waf_az_fw_nat_rule
- azure_waf_az_fw_network_rule
- azure_waf_az_fw_threat_intel
- azure_waf_azure_firewall_application_rule
- azure_waf_azure_firewall_network_rule

Gold tables:
- http_activity
- network_activity
- security_finding
