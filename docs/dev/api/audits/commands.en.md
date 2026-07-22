## /api/v1/terminal/commands/

### GET

- **Description:**
Query session command records. Returns the commands executed by the user in the asset session, their output, risk level and other information. It can be filtered by session, asset, command content, risk level and time range.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Remarks |
| --- | --- | --- |
| session_id | Type: string, session ID | Query command records in a specified session |
| asset_id | Type: String(UUID), asset ID | Query the command records on the specified asset |
| input | Type: String, command input | Filter by command content, fuzzy matching |
| risk_level | Type: Int, command risk level | Optional values: 0=accept, 4=warn, 5=reject, 6=review and reject, 7=review and accept, 8=review and cancel |
| date_from | Type: string(date-time), start time | For example: 2024-01-14T10:07:46.521Z |
| date_to | Type: string(date-time), end time | For example: 2024-01-22T15:59:59.000Z |
| search | Type: string, search term | Universal fuzzy search |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, command record list | The list elements are command record objects (see below) |
| id | Type: string, command record ID | UUID |
| user | Type: string, username | The user who executed the command |
| asset | Type: string, asset name | In the form `2.7(10.1.12.7)` |
| account | Type: string, account name | In the form of `root(root)` |
| input | Type: String, command input | The command actually executed by the user |
| output | Type: string, command output | Command execution echo content |
| session | Type: string, session ID | The session to which the command belongs |
| risk_level | Type: object, command risk level | In the form {"value":0,"label":"accept"} |
| org_id | Type: string, organization ID | UUID |
| timestamp | Type: int, execution timestamp | Unix timestamp (seconds) |
| timestamp_display | Type: string, execution time | In the form of `2024/01/19 19:05:40 +0800` |
| remote_addr | Type: string, remote address | User source IP |

- **Return Example:**

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "user": "mingliang.wang",
            "asset": "2.7(10.1.12.7)",
            "input": "docker ps -a",
            "session": "e37e1476-1a21-424a-909d-9f372c83e36b",
            "risk_level": {
                "value": 0,
                "label": "accept"
            },
            "org_id": "b495b355-b872-48a1-a8ab-615a9be7d49e",
            "id": "0e3e5f7b-4c4f-4711-9cb7-b44d654f7c7f",
            "account": "root(root)",
            "output": "docker ps -a\r\nCONTAINER ID   IMAGE                    COMMAND             CREATED       STATUS                 NAMES\r\n82d85c566fed   jumpserver/lion:v3.7.0   \"./entrypoint.sh\"   4 weeks ago   Up 4 weeks (healthy)   jms_lion",
            "timestamp": 1705662340,
            "timestamp_display": "2024/01/19 19:05:40 +0800",
            "remote_addr": "10.1.240.254"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?session_id=e37e1476-1a21-424a-909d-9f372c83e36b&offset=0&limit=15' \
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
SESSION_ID  = "your session id"

def get_commands():
    url = f"{API_URL}/api/v1/terminal/commands/"
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
        "session_id": SESSION_ID,
        "offset": 0,
        "limit": 15
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API Request failed:{e}")
        return None

if __name__ == "__main__":
    result = get_commands()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: Data was accidentally deleted on the database server in the early morning. The security auditor pulled command records containing `rm` based on assets and time ranges to locate who performed the deletion in which session.

```sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?asset_id=4bdae07e-c214-4a12-a9db-be8146219bc8&input=rm&date_from=2026-07-21T16:00:00.000Z&date_to=2026-07-22T04:00:00.000Z&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: One-click Audit of External Systems](../examples/one_click_audit.md)
