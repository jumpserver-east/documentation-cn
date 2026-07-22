## /api/v1/audits/login-logs/

### GET

- **Description:**
Query user login logs, that is, audit records of users logging into JumpServer through the Web, terminal, etc., including login source IP, city, authentication method, MFA status, and success or failure results.

> Note: This interface does not support the `date_from` / `date_to` time range parameters. Time filtering needs to be processed by the client based on the returned `datetime` field; it can be used with `order=-datetime` to pull pages in reverse order of login time and then intercept the required time period.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| username | Type: String, filter by username | - |
| ip | Type: String, filtered by login source IP | - |
| city | Type: String, filter by login city | - |
| type | Type: String, login source type | W（Web）/ T（Terminal）/ U（Unknown） |
| status | Type: int, login status | 1 (success) / 0 (failure) |
| mfa | Type: int, MFA status | 0 (disabled) / 1 (enabled) / 2 |
| id | Type: String(UUID), filter by log ID | - |
| search | Type: String, search term (user name, IP, etc. fuzzy matching can be filled in) | - |
| order | Type: string, sort field | Such as `datetime` (ascending order) / `-datetime` (descending order) |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, login log list | The list elements are login log objects (see below) |
| id | Type: string, log ID | UUID |
| username | Type: string, username | The format is like `name(username)` |
| type | Type: object, login source type | {"value":"W","label":"Web"} etc. |
| ip | Type: string, login source IP |    |
| city | Type: string, login city | The intranet address is displayed as `LAN` |
| user_agent | Type: string, user agent | Browser/Client UA information |
| mfa | Type: object, MFA level | {"value":0,"label":"disabled"} etc. |
| reason | Type: string, failure reason | Empty string when login is successful |
| reason_display | Type: string, failure reason description |    |
| backend | Type: string, authentication backend | Such as Password |
| backend_display | Type: string, authentication backend description | Such as password |
| status | Type: object, login status | {"value":true,"label":"success"} etc. |
| datetime | Type: string(date-time), login time | Client time filter based on this field |

- **Response example:**

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "19bf02fa-4cfd-4c03-8162-e62c20cf9505",
            "username": "skj(skj)",
            "type": {
                "value": "W",
                "label": "Web"
            },
            "ip": "10.1.240.254",
            "city": "LAN",
            "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
            "mfa": {
                "value": 0,
                "label": "Disable"
            },
            "reason": "",
            "reason_display": "",
            "backend": "Password",
            "backend_display": "Password",
            "status": {
                "value": true,
                "label": "success"
            },
            "datetime": "2024/01/22 10:17:20 +0800"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?offset=0&limit=15&order=-datetime' \
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

def get_login_logs():
    url = f"{API_URL}/api/v1/audits/login-logs/"
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
    result = get_login_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: The security group receives an alarm and investigates suspicious logins: Pull the failed login records of user zhangsan and page them in reverse order of login time. On the client, intercept the data of the last 24 hours based on the `datetime` field, and combine the source IP and the authentication backend to determine whether there is brute force cracking.

```sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?username=zhangsan&status=0&order=-datetime&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: Audit log docking with SIEM platform](../examples/siem_export.md)
