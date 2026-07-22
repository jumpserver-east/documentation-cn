## /api/v1/perms/users/{id}/assets/

### GET

- **Description:**
Query the list of assets authorized by the specified user

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Path parameters:**

| Parameter name | Type | Description | Is it required? |
| --- | --- | --- | --- |
| id | String | User ID, which can be obtained through `/api/v1/users/users/` interface query | Yes |

- **Query Parameters:**

| Parameter name | Type | Description | Is it required? |
| --- | --- | --- | --- |
| search | String | Search term (fuzzy match name/address) | No |
| name | String | Asset name | No |
| address | String | IP address | No |
| node_id | String | Node ID, filter authorized assets under this node | No |
| platform | String | System platform | No |
| category | String | category, such as host | No |
| type | String | Type, such as linux | No |
| is_active | Boolean | active state | No |
| limit | Int | Number of items displayed per page | No |
| offset | Int | paging offset | No |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, authorized asset list | List elements are asset objects (see below) |
| id | Type: string, asset ID | UUID |
| name | Type: string, asset name |    |
| address | Type: string, asset address | IP or domain name |
| platform | Type: object, system platform | {"id":1,"name":"Linux"} etc. |
| nodes | Type: list, belonging node |    |
| labels | Type: list, label |    |
| category | Type: object, category | {"value":"host","label":"host"} etc. |
| type | Type: object, type | {"value":"linux","label":"Linux"} etc. |
| connectivity | Type: object, connectability |    |
| is_active | Type: boolean, whether to activate |    |
| org_id | Type: string, organization ID |    |
| org_name | Type: String, organization name |    |
| date_created | Type: String(date-time), creation time |    |

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/perms/users/cf7a1f14-0c70-4209-8196-c24cb7ec41a4/assets/?offset=0&limit=15' \
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
USER_ID     = "cf7a1f14-0c70-4209-8196-c24cb7ec41a4"

def get_user_perm_assets(user_id):
    url = f"{API_URL}/api/v1/perms/users/{user_id}/assets/"
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

    try:
        response = requests.get(
            url, auth = auth, headers = headers
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API Request failed:{e}")
        return None

if __name__ == "__main__":
    result = get_user_perm_assets(USER_ID)
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: Before opening the bastion host entrance for engineer Zhang San (zhangsan), the operations platform first checks the scope of its currently authorized Linux production hosts and confirms that it only contains web assets and has no unauthorized authorization.

```sh
# Users of Zhang San ID Can pass first /api/v1/users/users/?username=zhangsan Query and get
curl -X GET 'https://localhost/api/v1/perms/users/3f2b9d6c-1a4e-4c58-9d2f-8e7a5b1c0d24/assets/?category=host&type=linux&search=web&is_active=true&limit=20&offset=0' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)
