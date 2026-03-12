# Microsoft Windows Security Event Log

Ingests Windows Security Event Log data (EVTX-derived JSONL) and normalizes it to OCSF gold tables for authentication, account change, and group management activity.

## Input Format

JSONL files produced by `pyevtx-rs` (or compatible exporters such as Winlogbeat/NXLog). Each line is a **double-encoded JSON string** — the outer string wraps the inner event JSON object:

```json
"{\"Event\":{\"System\":{...},\"EventData\":{\"Data\":[...]}}}"
```

The bronze `preTransform` handles the outer string unwrapping via `try_parse_json(data::STRING)`.

## Data Volume Path

Update the `inputs` path in `preset.yaml` to point to your Unity Catalog Volume:

```yaml
autoloader:
  inputs:
  - /Volumes/<catalog>/<schema>/<volume>/
```

Place JSONL files (extension `.jsonl`) in that Volume. The preset uses `pathGlobFilter: "*.jsonl"` to avoid scanning non-data files.

## Event IDs Covered

| Category | Event IDs | OCSF Class |
|---|---|---|
| Authentication | 4624, 4625, 4634, 4647, 4648, 4768, 4769, 4771 | 3002 — Authentication |
| Account Change | 4720, 4722, 4723, 4724, 4725, 4726, 4740 | 3001 — Account Change |
| Group Management | 4728, 4729, 4732, 4733, 4756, 4757 | 3006 — Group Management |

## Gold Tables

### `authentication` (OCSF class 3002)

| Field | Notes |
|---|---|
| `class_uid` | 3002 |
| `activity_id` / `activity_name` | 1=Logon, 2=Logoff, 3=Authentication Ticket |
| `status` | Success/Failure mapped from `keywords` hex flags (`0x8020…` = Audit Success) |
| `auth_protocol` / `auth_protocol_id` | NTLM=2, Kerberos=1, other=0 |
| `logon_type` / `logon_type_id` | Windows LogonType integer (2=Interactive, 3=Network, etc.) |
| `user` | `{name, domain, uid}` — target (authenticated) user |
| `actor.user` | `{name, domain, session_uid}` — subject (initiating) user |
| `src_endpoint` | `{hostname, ip, port}` — source workstation |
| `dst_endpoint` | `{hostname}` — destination computer |
| `metadata` | Product: Microsoft-Windows-Security-Auditing |
| `raw_data` | Full event JSON string |

### `account_change` (OCSF class 3001)

| Field | Notes |
|---|---|
| `activity_id` | 1=Create, 3=Update, 4=Delete, 8=Enable, 9=Disable, 12=Lock |
| `user` | `{name, domain, uid, account_name}` — target account |
| `actor.user` | Subject user who performed the change |

### `group_management` (OCSF class 3006)

| Field | Notes |
|---|---|
| `activity_id` | 2=Add Member, 4=Remove Member |
| `group` | `{name, domain, uid}` — the group being modified |
| `user` | `{uid, name}` — the member added/removed |
| `actor.user` | Subject user who performed the operation |

## Silver Tables

Three intermediate tables are produced before OCSF normalization:

- `windows_security_authentication` — logon/logoff/Kerberos events with all EventData fields
- `windows_security_account_change` — user account lifecycle events
- `windows_security_group_management` — security group membership changes

## Test Data Generation

Use `pyevtx-rs` to parse `.evtx` files, or generate synthetic JSONL with the helper script in `~/projects/evtxparser/generate_test_data.py` (produces all 21 target EventIDs).

## References

- [OCSF Schema Browser](https://schema.ocsf.io)
- [Windows Security Event Log Reference](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/security-auditing-overview)
- JIRA: FEIP-1834
