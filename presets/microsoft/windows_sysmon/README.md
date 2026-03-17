# Microsoft Windows System Monitor Log

Ingests Windows System Monitor (Sysmon) event log data (EVTX-derived JSONL) and normalizes it to OCSF gold tables for process, network, file system, DNS, and registry activity.

## Input Format

JSONL files produced by `pyevtx-rs` (or compatible exporters). Each line is a **double-encoded JSON string** — the outer string wraps the inner event JSON object:

```json
"{\"Event\":{\"System\":{\"EventID\":1,...},\"EventData\":{\"Image\":\"...\",\"CommandLine\":\"...\"}}}"
```

The bronze `preTransform` handles the outer string unwrapping via `try_parse_json(data::STRING)`.

> **Note on EventID format:** `pyevtx-rs` emits `EventID` as a plain integer. The bronze layer uses `COALESCE` to handle both the plain-integer format (real data) and the `{"#text": N}` wrapped format (some exporters).

## Data Volume Path

Update the `inputs` path in `preset.yaml` to point to your Unity Catalog Volume:

```yaml
autoloader:
  inputs:
  - /Volumes/<catalog>/<schema>/<volume>/
```

## Event IDs Covered

| Category | Event IDs | OCSF Class |
|---|---|---|
| Process | 1 (Create), 5 (Terminate), 6 (Driver Load), 7 (Image Load), 8 (CreateRemoteThread), 10 (ProcessAccess), 25 (Tampering) | 4007 — Process Activity |
| Network | 3 (Connection) | 4001 — Network Activity |
| File System | 11 (FileCreate), 15 (FileCreateStreamHash), 23 (FileDelete), 26 (FileDeleteDetected) | 1001 — File System Activity |
| DNS | 22 (DNS Query) | 4003 — DNS Activity |
| Registry Key | 12 (Create/Delete Key/Value), 14 (Rename) | 201001 — Registry Key Activity |
| Registry Value | 13 (SetValue) | 201002 — Registry Value Activity |

## Gold Tables

### `process_activity` (OCSF class 4007)

| Field | Notes |
|---|---|
| `class_uid` | 4007 |
| `activity_id` / `activity_name` | 1=Launch (Ev1), 2=Terminate (Ev5), 3=Open (Ev10), 4=Inject (Ev8), 99=Other (Ev6/7/25) |
| `process` | `{pid, file.path, cmd_line, user.name, integrity_info.level}` — primary process |
| `parent_process` | `{pid, file.path, cmd_line, user.name}` — parent (Event 1 only) |
| `actor.process` | Source process for injection events (Ev8/10) |
| `target.process` | Target process for injection events (Ev8/10) |
| `unmapped` | `rule_name`, `hashes`, `granted_access`, `call_trace`, `tamper_type`, `image_loaded`, `signed`, `signature_status` |

### `network_activity` (OCSF class 4001)

| Field | Notes |
|---|---|
| `activity_id` | 1=Open (all Event 3) |
| `src_endpoint` | `{ip, hostname, port}` |
| `dst_endpoint` | `{ip, hostname, port, svc_name}` |
| `connection_info` | `{direction_id, direction, protocol_name}` — Inbound/Outbound from `Initiated` field |
| `process` | `{pid, file.path, user.name}` — process that made the connection |

### `file_system_activity` (OCSF class 1001)

| Field | Notes |
|---|---|
| `activity_id` | 1=Create (Ev11/15), 4=Delete (Ev23/26) |
| `file` | `{path, hashes.{md5, sha256}}` — hashes extracted from `Hashes` or `Hash` field |
| `process` | `{pid, file.path, user.name}` |
| `unmapped` | `creation_utc_time`, `is_executable`, `archived`, `contents` (ADS content for Ev15) |

### `dns_activity` (OCSF class 4003)

| Field | Notes |
|---|---|
| `activity_id` | 1=Query (all Event 22) |
| `status_id` | 1=Success (QueryStatus=0), 2=Failure (non-zero status) |
| `query` | `{hostname, type='A', type_id=1}` |
| `answers` | Array of `{rdata}` parsed from semicolon-delimited `QueryResults` |
| `process` | `{pid, file.path, user.name}` |

### `registry_key_activity` (OCSF class 201001 — Windows extension)

| Field | Notes |
|---|---|
| `activity_id` | 1=Create (CreateKey/CreateValue), 2=Delete (DeleteKey/DeleteValue), 5=Rename (RenameKey) |
| `reg_key` | `{path}` — full registry path |
| `is_persistence_target` | `true` if path contains Run key, Winlogon, or Services paths |
| `unmapped` | `new_name` for rename events |

### `registry_value_activity` (OCSF class 201002 — Windows extension)

| Field | Notes |
|---|---|
| `activity_id` | 1=Set (all Event 13) |
| `reg_key` | `{path}` — parent key path (extracted from `TargetObject`) |
| `reg_value` | `{name, data}` — value name and data written |
| `is_persistence_target` | `true` if path contains Run key, Winlogon, or Services paths |

## Silver Tables

Six intermediate tables produced before OCSF normalization:

| Table | Event IDs |
|---|---|
| `windows_sysmon_process` | 1, 5, 6, 7, 8, 10, 25 |
| `windows_sysmon_network` | 3 |
| `windows_sysmon_file` | 11, 15, 23, 26 |
| `windows_sysmon_dns` | 22 |
| `windows_sysmon_registry_key` | 12, 14 |
| `windows_sysmon_registry_value` | 13 |

## Test Data

Real Sysmon samples were sourced from:
- [sbousseaden/EVTX-ATTACK-SAMPLES](https://github.com/sbousseaden/EVTX-ATTACK-SAMPLES)
- [Yamato-Security/hayabusa-sample-evtx](https://github.com/Yamato-Security/hayabusa-sample-evtx)
- [NextronSystems/evtx-baseline](https://github.com/NextronSystems/evtx-baseline)

Converted to JSONL with `pyevtx-rs`. Synthetic records covering all 16 target EventIDs are generated by `scripts/generate_sysmon_test_data.py` in the development project.

## References

- [OCSF Schema Browser](https://schema.ocsf.io)
- [Sysmon Event ID Reference](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [MITRE ATT&CK — Sysmon Detection](https://attack.mitre.org)
- JIRA: FEIP-1835
