## /api/v1/audits/password-change-logs/

### GET

- **Description:**
Query password change logs. This endpoint corresponds to the **Password Change Logs** page under **Audit**. It records changes to JumpServer user passwords, including the affected user, the person who changed the password, the source IP, and the change time. Asset account password rotation records belong to the **Account Password Change** feature; see the [Password Change Plan API](../pam/changepwd.md), which is outside the scope of this endpoint.

> Note: This endpoint does not support the `date_from` / `date_to` time-range parameters. The client must filter by the returned `datetime` field. Use `order=-datetime` to retrieve pages in reverse chronological order, then select the required time range.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| user | Type: String, filtered by users whose passwords have been changed | - |
| change_by | Type: String, filter by the person who changed the password (changer) | - |
| remote_addr | Type: String, filter by source IP | - |
| search | Type: String, search term (fuzzy matching such as user, password changer, IP, etc. can be filled in) | - |
| order | Type: string, sort field | Such as `datetime` (ascending order) / `-datetime` (descending order) |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, password change log list | The list elements are password change log objects (see below) |
| id | Type: string, log ID | UUID |
| user | Type: string, the user whose password was changed | The format is like `name(username)` |
| change_by | Type: string, password changer (changer) | User who performs password change operation |
| remote_addr | Type: string, source IP | Can be null |
| datetime | Type: string(date-time), password change time | Clients filter by time using this field |

- **Response example:**

> The examples are schematic data, and the fields are subject to online /api/docs.

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "3f7b9c2e-8d41-4a5b-9c6d-1e2f3a4b5c6d",
            "user": "Zhang San(zhangsan)",
            "change_by": "Administrator(admin)",
            "remote_addr": "10.1.240.254",
            "datetime": "2026/07/01 10:17:20 +0800"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/password-change-logs/?offset=0&limit=15&order=-datetime' \
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

def get_password_change_logs():
    url = f"{API_URL}/api/v1/audits/password-change-logs/"
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
        "offset": 0,
        "limit": 15,
        "order": "-datetime"
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
    result = get_password_change_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: Before the equal protection inspection, the security group needs to export the user password change records of this quarter as compliance certificates: You can use `search` to narrow the scope by user name (such as verifying the administrator admin), pull it in pages in reverse order of the password change time, and intercept the data of this quarter based on the `datetime` field on the client and archive it for retention.

```sh
curl -X GET 'https://localhost/api/v1/audits/password-change-logs/?search=admin&order=-datetime&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: Audit log docking with SIEM platform](../examples/siem_export.md)
