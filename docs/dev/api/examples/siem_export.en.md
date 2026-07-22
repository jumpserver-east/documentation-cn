# Practical case: Audit log docking with SIEM platform

## Scene description

Security compliance (classification protection, ISO 27001, internal audit, etc.) usually requires that the audit data of the bastion host be retained for a long time and can be correlated and analyzed, and the JumpServer itself has a limited log retention period. This case uses the API to continuously synchronize JumpServer's four types of audit data - user login logs, operation logs, session records, and command records - to local JSON Lines files in time increments, and then sends them to Splunk, ELK or self-built data warehouses through collectors such as Filebeat/Logstash to achieve long-term retention and unified retrieval of audit data. The synchronization script uses the local checkpoint file to record the last synchronization position, and only pulls the data in the incremental interval each time, which can be scheduled periodically through crontab.

## Preconditions

- JumpServer V4 environment, whose HTTPS address is accessible from the sync host on the network;
- An account with audit data viewing rights (system auditor or administrator role recommended), and has created an API Key (AK/SK) or obtained a Bearer Token (see the API document authentication chapter);
- Clarify the organizations to be synchronized: audit data is isolated by organization, and the request header `X-JMS-ORG` determines which organization's data to pull. Multi-organization environments need to be executed one by one;
- Install Python 3.7+ on the synchronization host and install dependencies: `pip install requests httpsig`;
- (Optional) Collector such as Filebeat/Logstash has been deployed to send the JSON Lines file output by the script to SIEM.

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/audits/login-logs/` | Get user login log |
| GET | `/api/v1/audits/operate-logs/` | Get operation log |
| GET | `/api/v1/terminal/sessions/` | Get session records |
| GET | `/api/v1/terminal/commands/` | Get command record |

The four interfaces are all standard paging interfaces (`limit` / `offset`, the response contains `count` / `next` / `previous` / `results`), but the supported time filtering parameters are different, and the incremental pull strategies are also different:

| interface | time increment mode |
| --- | --- |
| `/api/v1/terminal/commands/` | Server-side `date_from` / `date_to` (ISO 8601 time) precise filtering |
| `/api/v1/audits/operate-logs/`、`/api/v1/terminal/sessions/` | The server-side `days` parameter is roughly filtered by the number of days, and the client is precisely truncated by checkpoint. |
| `/api/v1/audits/login-logs/` | No server-side time filtering parameters: Use `order=-datetime` to turn pages in reverse order, and the client will truncate and terminate page turning early according to checkpoint. |

> Field-level details for each endpoint (query parameters and response fields) are available in the online documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

The following curl examples use Bearer Token authentication; all requests need to carry two request headers: `Authorization: Bearer <token>` and `X-JMS-ORG`.

### Step 1: Pull the login log

The login log interface does not have a time filter parameter, so use `order=-datetime` to turn pages in reverse order by login time: the latest data is listed first, and the client only retains records that are later than the checkpoint. Once all records on a page are earlier than the checkpoint, the page will stop turning to avoid full deep page turning. The key fields of returned records include `id`, `username`, `type` (login method), `ip`, `city`, `mfa`, `status`, `datetime` (login time), etc.

```sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?order=-datetime&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 2: Pull the operation log

The operation log interface supports the `days` parameter (coarse filtering based on the latest number of days). The length of time from the checkpoint to the current is rounded up to the number of days and passed in. The server first reduces the amount of data, and then cooperates with `order=-datetime` to accurately truncate the increment with the client in reverse order. The key fields of returned records are `id`, `user`, `action` (action), `resource_type`, `resource`, `remote_addr`, `org_name`, `datetime`, etc.

```sh
curl -X GET 'https://localhost/api/v1/audits/operate-logs/?days=1&order=-datetime&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 3: Pull session records

The session interface also supports `days` coarse filtering, and the incremental baseline uses the session start time `date_start`. Note that the session has a life cycle: `date_end` is empty when the session has not ended, and `is_finished` indicates whether it has ended. If the SIEM side cares about the session duration, it can be overwritten and updated by `id` after entering the database (the example in this article is incrementally exported once by `date_start`). Key fields include `id`, `user`, `asset`, `account`, `protocol`, `login_from`, `remote_addr`, `is_finished`, `date_start`, `date_end`, etc.

```sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?days=1&order=-date_start&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 4: Pull command records

Command records are the largest among the four types of data. Fortunately, the interface natively supports `date_from` / `date_to` (ISO 8601 format) server-side precise filtering, which can be directly passed into the `[checkpoint, now]` interval without client truncation. Note two points: the `output` (command output) field is base64 encoded and can be decoded on demand before entering the database; `timestamp` is a Unix second-level timestamp, and `timestamp_display` is the readable time. The remaining key fields include `id`, `user`, `asset`, `account`, `session` (the session ID, which can be associated with the session data in the third step), `input` (command content), `risk_level`, etc.

```sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?date_from=2026-07-21T00:00:00Z&date_to=2026-07-22T00:00:00Z&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 5: Write JSON Lines and advance checkpoint

Each type of data is written into a daily rolling `.jsonl` file (one JSON record per line, with an additional `_log_type` field identifying the data type) for direct collection by Filebeat/Logstash; each data source will advance its checkpoint to the right boundary of the current round window (script startup time) only after successful synchronization. The failure of one source will not affect other sources, and the next run will automatically make up for it. Output example (illustrative):

```json
{"id": "1c92cc2b-6b90-4749-a9bb-e5372a1c26c9", "username": "admin", "type": {"value": "W", "label": "Web"}, "ip": "203.0.113.10", "datetime": "2026/07/22 10:48:46 +0800", "_log_type": "login_logs"}
```

## Complete sample code

```python
# -*- coding: utf-8 -*-
# JumpServer Audit data incremental synchronization script：output JSON Lines，supply Filebeat/Logstash Collect in SIEM
# Depend on：pip install requests httpsig
# Usage：python3 siem_export.py（It is recommended to cooperate crontab Periodic execution）

import json
import math
import os
import sys
from datetime import datetime, timedelta, timezone

import requests
from httpsig.requests_auth import HTTPSignatureAuth

# ======================= Basic configuration =======================
API_URL    = "https://localhost"                        # JumpServer Access address
KEY_ID     = "your id"                                  # API Key ID（AK）
KEY_SECRET = "your secret"                              # API Key Secret（SK）
ORG_ID     = "00000000-0000-0000-0000-000000000002"     # organization ID，When there are multiple organizations, execute one by one.

PAGE_SIZE              = 100                            # paging size，It is not recommended to set it too large
DEFAULT_LOOKBACK_HOURS = 24                             # first run（None checkpoint）Lookback duration
CHECKPOINT_FILE        = "/opt/jms-siem/checkpoint.json"
OUTPUT_DIR             = "/opt/jms-siem/output"
VERIFY_SSL             = False                          # Self-signed certificate environment settings False

if not VERIFY_SSL:
    requests.packages.urllib3.disable_warnings()

# ======================= Authentication and Request =======================
def build_auth():
    """AK/SK Signature authentication；If used Bearer Token，Can be removed auth，Change to headers Add in Authorization"""
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = ['(request-target)', 'accept', 'date']
    )

def build_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Accept": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }

def api_get(path, params):
    response = requests.get(
        f"{API_URL}{path}", auth = build_auth(), headers = build_headers(),
        params = params, verify = VERIFY_SSL, timeout = 60
    )
    response.raise_for_status()
    return response.json()

# ======================= time processing =======================
def parse_dt(value):
    """Compatible JumpServer Common time formats：2026/07/22 10:48:46 +0800 with ISO 8601"""
    if not value:
        return None
    v = str(value).strip().replace("Z", "+0000")
    for fmt in ("%Y/%m/%d %H:%M:%S %z",
                "%Y-%m-%dT%H:%M:%S.%f%z",
                "%Y-%m-%dT%H:%M:%S%z"):
        try:
            return datetime.strptime(v, fmt)
        except ValueError:
            continue
    return None

def to_utc_str(dt):
    return dt.astimezone(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")

# ======================= Paging pull =======================
def fetch_pages(path, params):
    """Turn through all pages in order（Used with server-side time filtering）"""
    offset, results = 0, []
    while True:
        data = api_get(path, dict(params, limit = PAGE_SIZE, offset = offset))
        results.extend(data.get("results", []))
        if not data.get("next"):
            break
        offset += PAGE_SIZE
    return results

def fetch_desc_window(path, params, time_key, since, until):
    """
    Turn pages in reverse order，only keep (since, until] Records within the range；
    When the entire page of records is older than since early termination，Avoid deep page turning。
    """
    offset, results = 0, []
    while True:
        data = api_get(path, dict(params, limit = PAGE_SIZE, offset = offset))
        records = data.get("results", [])
        if not records:
            break
        all_older = True
        for r in records:
            dt = parse_dt(r.get(time_key))
            if dt is None:                  # Reserve conservatively when time cannot be parsed，Pass it to the downstream press id Remove duplicates
                all_older = False
                results.append(r)
                continue
            if dt <= since:
                continue
            all_older = False
            if dt <= until:
                results.append(r)
        if all_older or not data.get("next"):
            break
        offset += PAGE_SIZE
    return results

def coarse_days(since, until):
    """put checkpoint The length of time since now is converted into days parameters（Server-side coarse filtering，The client then accurately truncates）"""
    return max(1, math.ceil((until - since).total_seconds() / 86400))

# ======================= Four types of data sources =======================
def sync_login_logs(since, until):
    # Login log：The interface has no time filtering parameters，Turn pages in reverse order + client truncation
    return fetch_desc_window("/api/v1/audits/login-logs/",
                             {"order": "-datetime"},
                             "datetime", since, until)

def sync_operate_logs(since, until):
    # Operation log：days Coarse filter + Turn pages in reverse order + client truncation
    return fetch_desc_window("/api/v1/audits/operate-logs/",
                             {"days": coarse_days(since, until), "order": "-datetime"},
                             "datetime", since, until)

def sync_sessions(since, until):
    # Session recording：days Coarse filter，By session start time date_start Increment
    return fetch_desc_window("/api/v1/terminal/sessions/",
                             {"days": coarse_days(since, until), "order": "-date_start"},
                             "date_start", since, until)

def sync_commands(since, until):
    # Command record：date_from / date_to Server-side precise filtering
    return fetch_pages("/api/v1/terminal/commands/",
                       {"date_from": to_utc_str(since), "date_to": to_utc_str(until)})

# ======================= output with checkpoint =======================
def write_jsonl(log_type, records):
    if not records:
        print(f"[{log_type}] No new data this time")
        return
    os.makedirs(OUTPUT_DIR, exist_ok = True)
    day = datetime.now(timezone.utc).strftime("%Y%m%d")
    path = os.path.join(OUTPUT_DIR, f"jumpserver-{log_type}-{day}.jsonl")
    with open(path, "a", encoding = "utf-8") as f:
        for r in records:
            r["_log_type"] = log_type       # Convenient SIEM End-differentiated data types
            f.write(json.dumps(r, ensure_ascii = False) + "\n")
    print(f"[{log_type}] New {len(records)} Article -> {path}")

def load_checkpoints():
    if os.path.exists(CHECKPOINT_FILE):
        with open(CHECKPOINT_FILE, encoding = "utf-8") as f:
            return json.load(f)
    return {}

def save_checkpoints(cps):
    os.makedirs(os.path.dirname(CHECKPOINT_FILE), exist_ok = True)
    tmp = CHECKPOINT_FILE + ".tmp"
    with open(tmp, "w", encoding = "utf-8") as f:
        json.dump(cps, f, indent = 2)
    os.replace(tmp, CHECKPOINT_FILE)        # Atomic replacement，Avoid half-write corruption checkpoint

# ======================= Main process =======================
def main():
    cps = load_checkpoints()
    until = datetime.now(timezone.utc)      # Right border of current round window：Script startup time
    default_since = until - timedelta(hours = DEFAULT_LOOKBACK_HOURS)

    jobs = [
        ("login_logs",   sync_login_logs),
        ("operate_logs", sync_operate_logs),
        ("sessions",     sync_sessions),
        ("commands",     sync_commands),
    ]
    failed = []
    for name, func in jobs:
        since = parse_dt(cps.get(name)) or default_since
        try:
            records = func(since, until)
            write_jsonl(name, records)
            cps[name] = to_utc_str(until)   # The source succeeds only in advancing its own checkpoint
            save_checkpoints(cps)
        except Exception as e:
            failed.append(name)
            print(f"[{name}] Sync failed，checkpoint Not advanced，Automatic replenishment at next run：{e}",
                  file = sys.stderr)

    if failed:
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Deployment: crontab periodic scheduling

```sh
# every 5 Sync in minute increments，Append output to log file
*/5 * * * * /usr/bin/python3 /opt/jms-siem/siem_export.py >> /var/log/jms-siem-export.log 2>&1
```

### Deployment: Filebeat collection JSON Lines (example)

```yaml
filebeat.inputs:
  - type: filestream
    id: jumpserver-audit
    paths:
      - /opt/jms-siem/output/*.jsonl
    parsers:
      - ndjson:
          target: ""
          add_error_key: true
output.logstash:
  hosts: ["logstash.example.com:5044"]
```

## FAQ

**Q1: The amount of data is huge. What should I do if I need to turn through hundreds of pages in one synchronization? **

A: The core idea is to make each incremental window as small as possible instead of increasing the single page capacity. Shorten the crontab scheduling cycle (for example, once every 5 minutes), and keep `limit` around 100; command records support `date_from` / `date_to`, which can cut the large window into multiple small time periods and pull them in segments; the login/operation log relies on reverse page turning and early termination. The shorter the checkpoint interval, the shallower the page turning. Try to avoid deep page turning with large `offset`. The larger the database offset, the slower the query.

**Q2: What are the pitfalls of time format and time zone? **

A: The time returned by the JumpServer interface may be `2026/07/22 10:48:46 +0800`. This type of format with time zone offset may also be ISO 8601. It must include time zone parsing (such as `parse_dt` in the script) and internally convert it to UTC for comparison. Do not use local time without time zone for direct comparison, otherwise the checkpoint will miss data or be repeated. Values ​​passed to `date_from` / `date_to` use ISO 8601 with time zone (e.g. `2026-07-22T00:00:00Z`). The `timestamp` recorded by the command is a Unix second-level timestamp without time zone ambiguity, and `timestamp_display` is the formatted time.

**Q3: If the checkpoint file is lost, will re-running result in duplicate data? **

A: It will be pulled repeatedly, but it can be idempotent: the four types of records have a globally unique `id` (UUID), and the SIEM side uses `id` as the unique key to automatically deduplicate (Elasticsearch sets `id` to the document `_id` when writing, and Splunk uses `dedup id` when retrieving). After the checkpoint is lost, the script will only backtrack `DEFAULT_LOOKBACK_HOURS` (24 hours) by default. If you need to supplement earlier data, you can temporarily increase the value and run again. It is recommended to include the checkpoint file in the backup and keep the `.jsonl` file in the output directory for a period of time.

**Q4: The request returns 403, or the pulled data is incomplete? **

A: First check the account permissions - the audit interface requires auditor or administrator level viewing permissions, ordinary users can only see their own data (such as `my-login-logs`). Check `X-JMS-ORG` again: audit data is isolated by organization. This request header determines which organization's data is pulled. In a multi-organization environment, each organization ID must be traversed to perform synchronization separately, otherwise audit records of other organizations will be missed.
