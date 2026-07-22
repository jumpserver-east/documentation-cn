# Practical case: one-click audit of external systems

## Scene description

The enterprise's security operation/audit platform usually requires centralized auditing of the operating behavior of the bastion host: given "personnel + time period" (or "assets + time period"), all audit data under this dimension can be retrieved with one click - when and where the system was logged in, which assets were logged in, which commands were executed in the session, and summarized into an audit report. This case implements the "one-click audit" capability of external systems by arranging JumpServer's login logs, session records, command records and other audit APIs, and provides a Python script that can be run directly.

## Preconditions

- JumpServer V4 environment, external systems and JumpServer network are reachable;
- An API account with audit permissions: system auditor/system administrator role (or RBAC role with corresponding audit view permissions), ordinary users can only query their own records;
- Choose one of two authentication credentials:
    - API Key (AK/SK): Created in JumpServer "Personal Information - API Key", combined with `httpsig` signature authentication (recommended, long-term valid);
    - Bearer Token: The request header carries `Authorization: Bearer <token>`;
- Audit data is isolated by organization. You need to know the ID of the organization where the target data is located, which is specified through the request header `X-JMS-ORG` (leaving it blank defaults to the `Default` organization); cross-organization auditing needs to traverse the organization ID and call it separately;
- The complete example requires Python 3.7+ and installation dependencies: `pip install requests httpsig`;
- If JumpServer uses a self-signed certificate, the `requests` call can set `verify=False` as needed (it is recommended to configure a trusted certificate for production environments).

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/users/users/` | Query users by username and obtain `user_id` |
| GET | `/api/v1/audits/login-logs/` | Query the login log (when, where, and how to log in) |
| GET | `/api/v1/terminal/sessions/` | Query session records (which assets were logged in and what account was used) |
| GET | `/api/v1/terminal/commands/` | Query the command records in the session (which commands were executed) |
| GET | `/api/v1/terminal/sessions/{id}/replay/` | Get session recording file address |
| GET | `/api/v1/terminal/sessions/{id}/replay/download/` | Download session recording file |

> Note: Field-level details for these endpoints (all query parameters and response fields) are available in the JumpServer online API documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

All requests must carry a unified authentication request header:

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the format is fixed to `Bearer <token>`; when using AK/SK, it is changed to httpsig signature |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |

### Step 1: Locate the user by username and obtain user_id

The session recording interface is filtered by `user_id` (UUID), and the input of the audit platform is usually the user name, so the user name is first parsed into `user_id` through the `username` parameter of the user list interface.

``` sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

The `id` of the first element in the returned list is `user_id` (such as `d461c2e0-95cd-4ccd-aee8-a5d767560eea`).

### Step 2: Query the user’s login log

The login log interface records the user's login behavior to the JumpServer platform itself, and supports filtering by `username`, `ip`, `city`, `type` (W=Web / T=Terminal / U=Unknown), `status` (1=success / 0=failure), `search` and other conditions.

| Parameter name | Type | Description | Is it required? |
| --- | --- | --- | --- |
| username | String | Username | No |
| ip | String | Login source IP | No |
| status | Int | Login status, 1 successful / 0 failed | No |
| type | String | Login type, W/T/U | No |
| search | String | Search keywords | No |
| limit | Int | Number of items per page | No |
| offset | Int | paging offset | No |

``` sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?username=zhangsan&status=1&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

The `datetime` field of each record in the returned results is the login time (such as `2026/07/21 10:17:20 +0800`), and the filtering of the audit time period is completed by this field on the external system side (see the complete sample code).

### Step 3: Query the user’s session records (which assets have been logged in)

The session recording interface is the main clue for asset operation auditing: a session corresponds to a complete process of "a user uses a certain account and logs in to an asset using a certain protocol." Main filtering parameters supported:

| Parameter name | Type | Description | Is it required? |
| --- | --- | --- | --- |
| user_id | String | User ID, used when auditing by person | No |
| asset_id | String | Asset ID, used when auditing by asset | No |
| is_finished | Boolean | Whether it has ended, true/false | No |
| protocol | String | Protocol, such as ssh/rdp | No |
| login_from | String | Login source: ST/RT/WT/DT/VT | No |
| days | Number | Query the sessions in the last N days (coarse filtering on the server side) | No |
| limit | Int | Number of items per page | No |
| offset | Int | paging offset | No |

Audit by "person + time period":

``` sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?user_id=d461c2e0-95cd-4ccd-aee8-a5d767560eea&days=30&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

To audit by "asset + time period", just replace `user_id` with `asset_id`:

``` sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?asset_id=4bdae07e-c214-4a12-a9db-be8146219bc8&days=30&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Each session returned contains fields such as `id` (session ID, used to check the command in the next step), `asset`, `account`, `protocol`, `date_start`, `date_end`, `is_finished`, `has_command` (whether there is a command record), `command_amount` (the number of commands), `has_replay` (whether there is a recording). Precise time period filtering is done on the external system side by the `date_start` field, and the `days` parameter is only used for coarse filtering on the server side to reduce the amount of data.

### Step 4: Query the command details of the session

The command recording interface supports accurately pulling all commands in a single session by `session_id`, and also supports `date_from`/`date_to` (ISO 8601 format, colons in curl need to be escaped to `%3A`) filtering by time period:

| Parameter name | Type | Description | Is it required? |
| --- | --- | --- | --- |
| session_id | String | Session ID | No |
| asset_id | String(UUID) | Asset ID | No |
| user | String | Username | No |
| account | String | Asset account | No |
| input | String | Search by command content | No |
| risk_level | Int | Risk Level: 0 Accept / 4 Warning / 5 Reject / 6-8 Review Related | No |
| date_from | String(date-time) | start time | No |
| date_to | String(date-time) | end time | No |
| limit | Int | Number of items per page | No |
| offset | Int | paging offset | No |

``` sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?session_id=8608a7af-f1ee-4f84-bb8a-1384b71914f5&date_from=2026-07-01T00%3A00%3A00.000Z&date_to=2026-07-21T23%3A59%3A59.999Z&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Each command returned contains fields such as `input` (command input), `output` (command output), `risk_level` (risk level), `timestamp_display` (execution time), `remote_addr` (source address), etc., which can be directly written into the audit report.

### Step 5 (optional): Obtain session recording

For sessions where `has_replay` is true, you can obtain the recording file address through the API or download the recording file directly and save it as an attachment to the audit report:

``` sh
# Get video file address（Return JSON，file The field is the video file download address.）
curl -X GET 'https://localhost/api/v1/terminal/sessions/8608a7af-f1ee-4f84-bb8a-1384b71914f5/replay/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# Download video files directly
curl -X GET 'https://localhost/api/v1/terminal/sessions/8608a7af-f1ee-4f84-bb8a-1384b71914f5/replay/download/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -o replay_8608a7af.tar
```

For online playback, log in to the JumpServer Web console, find the corresponding session under **Audit - Session Audit**, and click **Playback**.

## Complete sample code

The following script enters "user name + audit time period" and automatically completes: parse user → pull login log → pull session list → pull command details session by session, and finally generate a JSON audit report file.

```python
# -*- coding: utf-8 -*-
"""
JumpServer One-click audit script
press "Username + time period" Pull audit data:
  1. Parse by username user_id
  2. Pull the user's login log(local press datetime Filter time period)
  3. Pull the user's session records(days Server-side coarse screening + local press date_start Fine screening)
  4. Pull command details session by session(Server date_from/date_to filter)
  5. Summary output JSON Audit report

Depend on: Python 3.7+, pip install requests httpsig
"""

import json
import sys
from datetime import datetime

import requests
from httpsig.requests_auth import HTTPSignatureAuth

# ======== Basic configuration(Modify according to actual environment) ========
API_URL    = "https://localhost"
KEY_ID     = "your id"       # API Key ID(personal information - API Key Created in)
KEY_SECRET = "your secret"   # API Key Secret
ORG_ID     = "00000000-0000-0000-0000-000000000002"  # organization ID, Default organization

# ======== Audit input(personnel + time period) ========
USERNAME  = "zhangsan"
DATE_FROM = "2026-07-01T00:00:00.000Z"   # Audit start time(ISO 8601)
DATE_TO   = "2026-07-21T23:59:59.999Z"   # Audit end time(ISO 8601)

PAGE_SIZE = 100


def build_auth():
    """AK/SK Signature authentication; If used Bearer Token, Can be removed auth,
    Add to request header Authorization: Bearer <token> That’s it"""
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = ['(request-target)', 'accept', 'date'],
    )


def build_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Accept": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form),
    }


def api_get_all(path, params=None):
    """press limit/offset Cycle through pages, Pull all data from the list interface"""
    params = dict(params or {})
    params["limit"] = PAGE_SIZE
    params["offset"] = 0
    results = []
    while True:
        try:
            resp = requests.get(
                f"{API_URL}{path}",
                auth = build_auth(),
                headers = build_headers(),
                params = params,
            )
            resp.raise_for_status()
        except requests.RequestException as e:
            print(f"API Request failed: {path}, Error: {e}")
            sys.exit(1)
        data = resp.json()
        results.extend(data.get("results", []))
        if not data.get("next"):     # next for null Indicates that the last page has been reached
            return results
        params["offset"] += PAGE_SIZE


def parse_dt(value):
    """Compatible JumpServer Returned time format(2026/07/21 10:17:20 +0800)with ISO 8601"""
    for fmt in ("%Y/%m/%d %H:%M:%S %z",
                "%Y-%m-%dT%H:%M:%S.%f%z",
                "%Y-%m-%dT%H:%M:%S%z"):
        try:
            return datetime.strptime(value, fmt)
        except (TypeError, ValueError):
            continue
    return None


def within(value, start, end):
    """Determine whether the time string falls within the audit time period"""
    dt = parse_dt(value)
    return dt is not None and start <= dt <= end


def main():
    start = parse_dt(DATE_FROM)
    end = parse_dt(DATE_TO)
    if not start or not end:
        print("DATE_FROM / DATE_TO Format error, should be ISO 8601, Such as 2026-07-01T00:00:00.000Z")
        sys.exit(1)

    # 1. Parse by username user_id
    users = api_get_all("/api/v1/users/users/", {"username": USERNAME})
    if not users:
        print(f"User not found: {USERNAME}")
        sys.exit(1)
    user = users[0]
    user_id = user["id"]
    print(f"[1/4] Target users: {user.get('name')}({user.get('username')}), id={user_id}")

    # 2. Login log: Server press username filter, Press the time period locally datetime Field filtering
    login_logs = api_get_all("/api/v1/audits/login-logs/", {"username": USERNAME})
    login_logs = [x for x in login_logs if within(x.get("datetime"), start, end)]
    print(f"[2/4] Login log within time period {len(login_logs)} Article")

    # 3. Session recording: Server press user_id + days Coarse sieve, local press date_start Fine screening
    days = max((datetime.now(start.tzinfo) - start).days + 1, 1)
    sessions = api_get_all("/api/v1/terminal/sessions/",
                           {"user_id": user_id, "days": days})
    sessions = [x for x in sessions if within(x.get("date_start"), start, end)]
    print(f"[3/4] session within time period {len(sessions)} a")

    # 4. Pull command details session by session(Server support session_id + date_from/date_to filter)
    session_reports = []
    for s in sessions:
        commands = []
        if s.get("has_command"):     # graphics/SFTP Wait for the session to have no command records, skip
            commands = api_get_all("/api/v1/terminal/commands/", {
                "session_id": s["id"],
                "date_from": DATE_FROM,
                "date_to": DATE_TO,
            })
        session_reports.append({
            "session_id": s["id"],
            "asset": s.get("asset"),
            "account": s.get("account"),
            "protocol": s.get("protocol"),
            "remote_addr": s.get("remote_addr"),
            "date_start": s.get("date_start"),
            "date_end": s.get("date_end"),
            "is_finished": s.get("is_finished"),
            "has_replay": s.get("has_replay"),
            # Recorded sessions, You can use this address to download the video file as an audit attachment.
            "replay_download": (
                f"{API_URL}/api/v1/terminal/sessions/{s['id']}/replay/download/"
                if s.get("has_replay") else None
            ),
            "command_count": len(commands),
            "commands": [
                {
                    "input": c.get("input"),
                    "output": (c.get("output") or "")[:200],  # Truncate output, Avoid reporting too large
                    "risk_level": (c.get("risk_level") or {}).get("label"),
                    "timestamp_display": c.get("timestamp_display"),
                    "remote_addr": c.get("remote_addr"),
                }
                for c in commands
            ],
        })
    print("[4/4] Command details fetch completed")

    # 5. Summary audit report
    report = {
        "audit_target": {"username": USERNAME, "user_id": user_id},
        "audit_range": {"date_from": DATE_FROM, "date_to": DATE_TO},
        "generated_at": datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%SZ"),
        "summary": {
            "login_count": len(login_logs),
            "session_count": len(session_reports),
            "command_count": sum(x["command_count"] for x in session_reports),
        },
        "login_logs": [
            {
                "datetime": x.get("datetime"),
                "ip": x.get("ip"),
                "city": x.get("city"),
                "type": (x.get("type") or {}).get("label"),
                "status": (x.get("status") or {}).get("label"),
                "backend": x.get("backend_display"),
            }
            for x in login_logs
        ],
        "sessions": session_reports,
    }

    out_file = f"audit_report_{USERNAME}.json"
    with open(out_file, "w", encoding="utf-8") as f:
        json.dump(report, f, ensure_ascii = False, indent = 2)
    print(f"Audit report has been generated: {out_file}")
    print("Summary: " + json.dumps(report["summary"], ensure_ascii = False))


if __name__ == "__main__":
    main()
```

When auditing by "asset + time range", replace the step 3 filter `{"user_id": user_id}` with `{"asset_id": "<asset-ID>"}` (retrieve the asset ID by name or IP through the asset list endpoint). The rest of the process is unchanged.

## FAQ

**Q1: ​​Calling the audit interface returns 403, or the required data cannot be found? **

A: The audit interface requires the system auditor/system administrator role (or RBAC role with audit viewing permissions). Ordinary users can only query their own records. In addition, the audit data is isolated by organization, and the request header `X-JMS-ORG` must point to the ID of the organization where the data is located; when the user has operation records in multiple organizations, he needs to traverse the organization IDs and call them separately before merging them.

**Q2: How to filter session records and login logs by time period? **

A: In the interface definition, `/api/v1/terminal/commands/` supports `date_from`/`date_to` server-side filtering (ISO 8601 format, colons in curl need to be escaped to `%3A`); `/api/v1/terminal/sessions/` does not define these two parameters, you can first use the `days` parameter to press "Last N" Day" coarse filter, and then fine filter by the returned `date_start` field on the external system side; filter by `username` and other conditions in `/api/v1/audits/login-logs/`, then filter the time period by the `datetime` field on the external system side. The complete sample code is handled this way.

**Q3: Why can’t commands be found or videos downloaded in some sessions? **

A: Only command-line sessions (such as SSH and database CLI sessions) generate command records. For file or graphical sessions such as SFTP and RDP, `has_command` is false and the command count is 0. Recordings exist only when `has_replay`/`can_replay` is true. Recordings cannot be downloaded for unfinished sessions (`is_finished=false`) or when the component does not support recording. For online playback, click **Playback** for the corresponding session under **Audit - Session Audit** in the Web console.

**Q4: How many pieces of data can the list interface return at most at one time? **

A: All list interfaces are paginated uniformly, returning the `count/next/previous/results` structure. Page turning is controlled through `limit` (number of items per page) and `offset` (offset). `next` is null to indicate that the last page has been reached. In the one-click audit scenario, you must cycle through pages to pull all the data like the sample script. Only fetching the first page will cause audit data to be missing.
