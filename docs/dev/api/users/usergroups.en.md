## /api/v1/users/groups/

### POST

- **Description:**
Create user group

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Request body parameters (Body):**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| name* | Type: string, name | - |
| users | Type: object[], user; each element contains id (string), name (string), is_service_account (boolean) | - |

> Note: Parameters with * are required.
- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, user group id |    |
| name | Type: String, user group name |    |
| comment | Type: string, remarks |    |
| created_by | Type: string, creator |    |
| users | Type: object[], user; each element contains id (string), name (string), is_service_account (boolean) |    |
| org_id | Type: String, organization |    |
| org_name | Type: String, organization name |    |
| date_created | Type: String, creation time |    |

- **Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/users/groups/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{"name":"api_test"}'
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

def create_users_group():
    url = f"{API_URL}/api/v1/users/groups/"
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
    
    data = {
        "name": "api_test"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data),
            verify = False
        )
        response.raise_for_status()
        print("User group created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_users_group()
```

- **Use Case:**

Scenario: The company has newly formed a database operations team. The administrator creates the `dba-team` user group and directly adds members zhangsan and lisi to the group when creating it, which facilitates subsequent batch authorization of database assets by group.

```sh
curl -X POST 'https://localhost/api/v1/users/groups/' \
    -H 'Authorization: Bearer <token>' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "dba-team",
        "users": [
            {"id": "0f6a3fea-1b8f-4e4a-9b8e-2f4d5c6a7b8c", "name": "zhangsan"},
            {"id": "9c2d4e6f-3a5b-4c7d-8e9f-1a2b3c4d5e6f", "name": "lisi"}
        ]
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)

## /api/v1/users/groups/{id}/

### DELETE

- **Description:**
Delete user group

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/users/groups/7413d36d-cf37-45f4-b42a-cd5eeff1f4ff/' \
    -H 'Content-Type:application/json' \
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
GROUP_ID    = "your group id"

def delete_users_group():
    url = f"{API_URL}/api/v1/users/groups/{GROUP_ID}/"
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
        response = requests.delete(
            url, auth = auth, headers = headers,
            verify = False
        )
        response.raise_for_status()
        print("User group deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_users_group()
```

- **Use Case:**

Scenario: After the acceptance of the outsourced on-site project is completed, the administrator deletes the temporary user group `outsource-temp` (the group ID has been queried and obtained in advance), and promptly recovers the asset authorization associated with the group to avoid residual permissions.

```sh
curl -X DELETE 'https://localhost/api/v1/users/groups/5e8b2c1d-4f6a-4b3c-9d2e-7a8b9c0d1e2f/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)
