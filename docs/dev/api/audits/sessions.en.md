## /api/v1/terminal/sessions/

### GET

- **Description:**
Query session records (asset session audit), support filtering by user, asset, protocol, login source, time range (number of recent days) and other conditions

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional values / remarks |
| --- | --- | --- |
| user_id | Type: string, user ID | UUID |
| asset_id | Type: string, asset ID | UUID |
| protocol | Type: string, protocol | Such as ssh / rdp / sftp / mysql, etc. |
| login_from | Type: string, login source | ST=SSH Terminal / RT=RDP Terminal / WT=Web Terminal / DT=DB Terminal / VT=VNC Terminal |
| is_finished | Type: boolean, whether the session has ended | true / false; false to filter online sessions |
| days | Type: number, last N days | For example, days=7 means only the sessions within the last 7 days will be returned. |
| search | Type: string, search term | Support fuzzy search for username, assets, etc. |
| order | Type: string, sort field | Such as `-date_start` in reverse order by start time |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

> Note: This interface does not support `date_from` / `date_to` precise time period parameters (old data before V4 is incorrect). If you need to filter by precise time period, you can first use `days` to narrow the range, and then filter by the returned `date_start` field on the client.

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, session data list | List elements are session objects (see below) |
| id | Type: string, session ID | UUID |
| user | Type: string, user | The format is `name(username)` |
| user_id | Type: string, user ID | UUID |
| asset | Type: string, asset | The format is `name(address)` |
| asset_id | Type: string, asset ID | UUID |
| account | Type: string, account | The format is `name(username)` |
| account_id | Type: string, account ID | UUID |
| protocol | Type: string, protocol | ssh/rdp/sftp etc. |
| type | Type: object, session type | In the form {"value":"sftp","label":"SFTP"} |
| login_from | Type: object, login source | In the form {"value":"WT","label":"Web Terminal"} |
| remote_addr | Type: string, remote address | User source IP |
| comment | Type: string, remarks | Can be null |
| terminal_display | Type: string, component display name | The terminal component that handles this session |
| terminal | Type: object, terminal component | Contains id / name / type (such as koko) |
| is_locked | Type: boolean, whether it is locked |    |
| is_success | Type: boolean, whether the connection is successful or not |    |
| is_finished | Type: boolean, whether it has ended | false means online session |
| has_replay | Type: boolean, whether there is video recording |    |
| has_command | Type: boolean, whether there is a command record |    |
| can_replay | Type: boolean, whether it can be played back |    |
| can_join | Type: boolean, whether to join (collaboration) |    |
| can_terminate | Type: boolean, whether it can be interrupted |    |
| command_amount | Type: int, number of commands |    |
| error_reason | Type: object, error reason | In the form of {"value":"replay_unsupported","label":"Recording is not supported"} |
| org_id | Type: string, organization ID | UUID |
| org_name | Type: string, organization name |    |
| date_start | Type: string(date-time), start time |    |
| date_end | Type: string(date-time), end time | null if not ended |

- **Return Example:**

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "8608a7af-f1ee-4f84-bb8a-1384b71914f5",
            "user": "xx(jingyu.qi)",
            "asset": "linux-10.1.12.28(10.1.12.28)",
            "user_id": "d461c2e0-95cd-4ccd-aee8-a5d767560eea",
            "asset_id": "4bdae07e-c214-4a12-a9db-be8146219bc8",
            "account": "root(root)",
            "account_id": "cbc45cdb-0e4e-465a-ae73-0b5978240f86",
            "protocol": "sftp",
            "type": {
                "value": "sftp",
                "label": "SFTP"
            },
            "login_from": {
                "value": "WT",
                "label": "Web Terminal"
            },
            "remote_addr": "10.1.240.254",
            "comment": null,
            "terminal_display": "[KoKo]-jms1-22fcb8289055",
            "is_locked": false,
            "command_amount": 0,
            "error_reason": {
                "value": "replay_unsupported",
                "label": "Video recording is not supported"
            },
            "terminal": {
                "id": "82e34994-a778-47d7-ad3f-4ee5e9385cf7",
                "name": "[KoKo]-jms1-22fcb8289055",
                "type": "koko"
            },
            "org_id": "b495b355-b872-48a1-a8ab-615a9be7d49e",
            "org_name": "JS",
            "is_success": true,
            "is_finished": true,
            "has_replay": false,
            "has_command": false,
            "can_replay": false,
            "can_join": false,
            "can_terminate": false,
            "date_start": "2024/01/21 16:47:05 +0800",
            "date_end": "2024/01/21 16:47:44 +0800"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?days=7&is_finished=true&offset=0&limit=15' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

**Python**

```python
# Python Example

import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"
KEY_ID      = "your id"
KEY_SECRET  = "your secret"
ORG_ID      = "your org id"

def get_sessions():
    url = f"{API_URL}/api/v1/terminal/sessions/"
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    signature_headers = ['(request-target)', 'accept', 'date']
    headers = {
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }
    auth = HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = signature_headers
    )
    params = {
        "days": 7,
        "is_finished": "true",
        "offset": 0,
        "limit": 15
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers, params = params
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API Request failed:{e}")
        return None

if __name__ == "__main__":
    result = get_sessions()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: During the quarterly security audit, the auditor needs to pull a list of ended sessions that accessed the core database assets through the Web Terminal in the last 7 days, and check whether there is any abnormal access during non-working hours.

```sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?days=7&login_from=WT&asset_id=4bdae07e-c214-4a12-a9db-be8146219bc8&is_finished=true&order=-date_start&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: One-click Audit of External Systems](../examples/one_click_audit.md)

## /api/v1/terminal/sessions/{id}/replay/

### GET

- **Description:**
Get the recording information of the specified session (return the download address of the recording file). If you need to directly download the recording file, you can access the `/api/v1/terminal/sessions/{id}/replay/download/` interface to obtain the file stream. Only sessions with `has_replay` set to true are available for recording.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The response body is in JSON format (the download interface returns a file stream) |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string(UUID), session ID (can be obtained from the return of the session record list interface) | Yes |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| file | Type: string(uri), video file download address | Unable to obtain when the session has no recording (such as has_replay=false) |

> Note: `/replay/` returns the address information of the recording file; `/replay/download/` directly returns the recording file stream, which is suitable for offline evidence collection or importing the playback tool after the script is saved to disk.

- **Request Example**

**CURL**

``` sh
# Get video information
curl -X GET 'https://localhost/api/v1/terminal/sessions/SESSION_ID/replay/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# Download video file（Save to local）
curl -X GET 'https://localhost/api/v1/terminal/sessions/SESSION_ID/replay/download/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -o 'SESSION_ID.replay.gz'
```

**Python**

```python
# Python Example

import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"
KEY_ID      = "your id"
KEY_SECRET  = "your secret"
ORG_ID      = "your org id"
SESSION_ID  = "your session id"

def build_auth_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    signature_headers = ['(request-target)', 'accept', 'date']
    headers = {
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }
    auth = HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = signature_headers
    )
    return auth, headers

def get_replay_info():
    url = f"{API_URL}/api/v1/terminal/sessions/{SESSION_ID}/replay/"
    auth, headers = build_auth_headers()

    try:
        response = requests.get(
            url, auth = auth, headers = headers
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API Request failed:{e}")
        return None

def download_replay(save_path):
    url = f"{API_URL}/api/v1/terminal/sessions/{SESSION_ID}/replay/download/"
    auth, headers = build_auth_headers()

    try:
        response = requests.get(
            url, auth = auth, headers = headers, stream = True
        )
        response.raise_for_status()
        with open(save_path, "wb") as f:
            for chunk in response.iter_content(chunk_size = 8192):
                f.write(chunk)
        print(f"Video has been saved: {save_path}")
    except requests.RequestException as e:
        print(f"API Request failed:{e}")

if __name__ == "__main__":
    info = get_replay_info()
    print(json.dumps(info, indent = 2, ensure_ascii = False))
    download_replay(f"{SESSION_ID}.replay.gz")
```

- **Use Case:**

Scenario: During the audit, it is discovered that a certain session executed a high-risk command. The auditor downloads the recording file of the session based on the session ID and archives it to the forensic directory for offline playback analysis.

```sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/8608a7af-f1ee-4f84-bb8a-1384b71914f5/replay/download/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -o '/data/audit/replays/8608a7af.replay.gz'
```

> For complete integration scenarios, please refer to: [Practical Case: One-click Audit of External Systems](../examples/one_click_audit.md)
