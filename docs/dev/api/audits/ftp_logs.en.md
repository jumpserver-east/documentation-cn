## /api/v1/audits/ftp-logs/

### GET

- **Description:**
Query file transfer records corresponding to the **File Transfer** page under **Audit**. These records audit users' file uploads and downloads on assets through SFTP, RDP, and other protocols, including the user, source address, asset, account, file name, action, and result.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| user | Type: String, filter by operating user | - |
| asset | Type: String, filter by asset | - |
| account | Type: String, filter by login account | - |
| filename | Type: String, filter by file name | - |
| session | Type: String, filtered by session ID | - |
| search | Type: String, search term (fuzzy matching of user, asset, file name, etc. can be filled in) | - |
| order | Type: string, sort field | Such as `date_start` (ascending order) / `-date_start` (descending order) |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, file transfer record list | The list elements are file transfer record objects (see below) |
| id | Type: string, record ID | UUID |
| user | Type: string, operating user | The format is like `name(username)` |
| remote_addr | Type: string, source address | User client IP, optionally null |
| asset | Type: string, asset | The format is like `name(address)` |
| account | Type: string, login account | Account used to log in to assets |
| org_id | Type: string, organization ID |    |
| operate | Type: object, operation action | {"value":..,"label":..}, such as upload/download/delete, etc. |
| filename | Type: string, file name | The path/name of the file on the asset |
| date_start | Type: string(date-time), operation time | read-only fields |
| is_success | Type: boolean, whether it was successful or not |    |
| has_file | Type: boolean, whether the file can be downloaded | true means the file was retained and can be downloaded from the Audit page |
| session | Type: string, the session ID to which it belongs | Associate online/historical sessions |

- **Response example:**

> The examples are schematic data, and the fields are subject to online /api/docs.

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "5f0f6b52-7a9c-4c3a-9d2e-8b1a2c3d4e5f",
            "user": "Zhang San(zhangsan)",
            "remote_addr": "10.1.240.254",
            "asset": "prod-web-01(192.168.1.10)",
            "account": "root",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "operate": {
                "value": "download",
                "label": "Download"
            },
            "filename": "/data/app/config/app.properties",
            "date_start": "2026-07-20 10:17:20 +0800",
            "is_success": true,
            "has_file": true,
            "session": "9c8b7a6d-5e4f-4a3b-8c1d-0e9f8a7b6c5d"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/ftp-logs/?offset=0&limit=15&order=-date_start' \
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

def get_ftp_logs():
    url = f"{API_URL}/api/v1/audits/ftp-logs/"
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
        "order": "-date_start"
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
    result = get_ftp_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: A security audit found that production data was suspected to be leaked. The auditor pulled the file transfer records of user zhangsan on the production server prod-web-01 and arranged them in reverse order of operation time. The client filtered out the "download" action based on the returned `operate` field and checked which files it had recently downloaded.

```sh
curl -X GET 'https://localhost/api/v1/audits/ftp-logs/?user=zhangsan&asset=prod-web-01&order=-date_start&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: One-click Audit of External Systems](../examples/one_click_audit.md)
