## /api/v1/audits/operate-logs/

### GET

- **Description:**
Query operation logs corresponding to the **Operation Logs** page under **Audit**. They record create, delete, update, and other actions performed in the JumpServer Web console, including the user, action type, resource type, resource name, source IP, and operation time.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| action | Type: String, filter by action | view/update/delete/create/export/download/connect/login/change_password/accept/review/notice/reject/approve/close/finished |
| days | Type: number, the number of recent days | For example, `7` means the logs in the last 7 days |
| days__lt | Type: number, upper bound of days (earlier than the last N days) | - |
| user | Type: String, filter by operating user | - |
| resource | Type: String, filter by resource name | - |
| resource_type | Type: String, filter by resource type | Resource type names displayed on pages such as `Asset Authorization`, `User`, etc. |
| remote_addr | Type: String, filtered by operation source IP | - |
| search | Type: String, search term (fuzzy matching of users, resources, etc.) | - |
| order | Type: string, sort field | Such as `datetime` (ascending order) / `-datetime` (descending order) |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, operation log list | The list elements are operation log objects (see below) |
| id | Type: string, log ID | UUID |
| user | Type: string, operating user | The format is like `name(username)` |
| action | Type: object, operation action | {"value":"update","label":"update"} etc. |
| resource_type | Type: string, resource type | Such as `asset authorization`, `user` |
| resource | Type: string, resource name | Display name of the operated object |
| remote_addr | Type: string, operation source IP | Can be null |
| org_id | Type: string, organization ID |    |
| org_name | Type: string, organization name |    |
| datetime | Type: string(date-time), operation time |    |

- **Response example:**

> The examples are schematic data, and the fields are subject to online /api/docs.

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "6f2b8a1c-9d3e-4f5a-b7c8-0a1b2c3d4e5f",
            "user": "Zhang San(zhangsan)",
            "action": {
                "value": "update",
                "label": "update"
            },
            "resource_type": "Asset authorization",
            "resource": "operations group-Production server licensing",
            "remote_addr": "10.1.240.254",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "org_name": "Default",
            "datetime": "2026/07/21 14:32:08 +0800"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/operate-logs/?offset=0&limit=15&order=-datetime' \
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

def get_operate_logs():
    url = f"{API_URL}/api/v1/audits/operate-logs/"
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
        "order": "-datetime",
        "limit": 15,
        "offset": 0
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
    result = get_operate_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: A security audit finds that the asset scope of an asset authorization rule has been expanded. It is necessary to trace who modified this authorization rule and when: filter update actions by resource type "asset authorization", use the rule name as a search term, and view the operating user and source IP in the modification record in reverse chronological order.

```sh
curl -X GET 'https://localhost/api/v1/audits/operate-logs/?action=update&resource_type=Asset authorization&search=operations group-Production server licensing&order=-datetime&limit=20' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: Audit log docking with SIEM platform](../examples/siem_export.md)
