## /api/v1/perms/asset-permissions/

### POST
- **Description:**
Create asset authorization

- **Request headers:**  

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name* | Type: String, name | - |
| users | Type: String[], user id | - |
| user_groups | Type: String[], user group id | - |
| assets | Type: String[], asset id | - |
| nodes | Type: String[], node id | - |
| accounts | Type: String[], account id, optional special values: <br>- "@ALL": all accounts <br>- "@SPEC": specified account <br>- "@INPUT": manual account <br>- "@USER": account with the same name | - |
| actions | Type: String[], action, optional values: [connect, upload, download, copy, paste, delete, share] | connect, upload, download, copy, paste, delete, share |
| is_active | Type: Boolean, activated | true |
| date_start | Type: String(datetime), start time | - |
| date_expired | Type: String(datetime), expiration time | - |
| comment | Type: String, remarks | - |

> Note: Parameters with * are required.
- **Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
-H 'Content-Type:application/json' \
-H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
-H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
-d '{
        "name":"create_asset_permission",
        "user_groups":["745980b1-54be-4c5e-b6ab-89826c2e2054"],
        "nodes":["3728f004-99a2-4fca-9577-84d5ffcf9eff"],
        "accounts":["@ALL"],
        "actions":["connect","upload","download","copy","paste"],
        "is_active":true,
        "date_start":"2025-01-01T00:00:00.879Z",
        "date_expired":"2095-01-01T00:00:00.879Z"
    }'
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
NODE_ID     = "your node id"

def create_asset_permissions():
    url = f"{API_URL}/api/v1/perms/asset-permissions/"
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
        "name":"create_asset_permission",
        "user_groups":[GROUP_ID],
        "nodes":[NODE_ID],
        "accounts":["@ALL"],
        "actions":["connect","upload","download","copy","paste"],
        "is_active": True,
        "date_start":"2025-01-01T00:00:00.879Z",
        "date_expired":"2095-01-01T00:00:00.879Z"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Asset authorization created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_asset_permissions()
```

- **Return parameters:**  

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| users | Type: String[], user |    |
| user_groups | Type: String[], user group |    |
| assets | Type: String[], asset |    |
| nodes | Type: String[], node |    |
| accounts | Type: String[], authorized account |    |
| actions | Type: String[], action |    |
| is_active | Type: Boolean, activated |    |
| created_by | Type: String, Creator |    |
| comment | Type: String, remarks |    |
| date_created | Type: String(date-time), authorization rule creation time |    |
| date_start | Type: String(date-time), authorization rule start time |    |
| date_expired | Type: String(date-time), authorization rule expiration time |    |

- **Use Case:**

Scenario: New employee zhangsan joins the operations group. The administrator separately authorizes the two assets web-server-01 and web-server-02 for him. Only accounts with the same user name are allowed to connect (no file transfer is allowed). The authorization automatically expires after 90 days.

```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer <token>' \
-H 'X-JMS-ORG: <organizationID>' \
-d '{
        "name": "zhangsan-web-servers-90d",
        "users": ["8b1f0a2e-3c4d-4e5f-9a6b-7c8d9e0f1a2b"],
        "assets": ["1a2b3c4d-5e6f-4a7b-8c9d-0e1f2a3b4c5d", "5d4c3b2a-1f0e-4d9c-8b7a-6f5e4d3c2b1a"],
        "accounts": ["@USER"],
        "actions": ["connect"],
        "is_active": true,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2026-10-20T00:00:00.000Z",
        "comment": "new employee zhangsan Onboarding authorization，Only connections with accounts with the same name are allowed，90 Expires after days"
    }'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)

## /api/v1/perms/asset-permissions/{id}/

### DELETE
- **Description:**
Delete asset authorization

- **Request headers:**  

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Path Params:**  

| Name | Description | Required |
| --- | --- | --- |
| id | Type: String, authorization ID | Yes |

- **Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/perms/asset-permissions/b2c45ce6-6b9c-4270-a073-567ff0e0ade8/' \
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
PER_ID      = "your permission id"

def delete_assets_permissions():
    url = f"{API_URL}/api/v1/perms/asset-permissions/{PER_ID}/"
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
            url, auth = auth, headers = headers
        )
        response.raise_for_status()
        print("Asset authorization deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_assets_permissions()
```

- **Use Case:**

Scenario: The project that outsourcer wangwu participated in has been delivered for acceptance, and the security audit requires that his temporary asset authorization be recovered on the same day (authorization ID is f3d2c1b0-a9e8-4d76-b543-2f1e0d9c8b7a).

```sh
curl -X DELETE 'https://localhost/api/v1/perms/asset-permissions/f3d2c1b0-a9e8-4d76-b543-2f1e0d9c8b7a/' \
-H 'Authorization: Bearer <token>' \
-H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)

### PUT
- **Description:**
Update asset authorization

- **Request headers:**  

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name* | Type: String, name | - |
| users | Type: String[], user id | - |
| user_groups | Type: String[], user group id | - |
| assets | Type: String[], asset id | - |
| nodes | Type: String[], node id | - |
| accounts | Type: String[], account number, optional special values: <br>- "@ALL": all accounts <br>- "@SPEC": specified account <br>- "@INPUT": manual account <br>- "@USER": account with the same name | - |
| actions | Type: String[], action, optional values: [connect, upload, download, copy, paste, delete, share] | connect, upload, download, copy, paste, delete, share |
| is_active | Type: Boolean, activated | true |
| date_start | Type: String(datetime), start time | - |
| date_expired | Type: String(datetime), expiration time | - |
| comment | Type: String, remarks | - |

> Note: Parameters with * are required.
- **Request Example**

**CURL**
```sh
curl -X PUT 'https://localhost/api/v1/perms/asset-permissions/ca90421b-bc45-48c0-bd49-4a213ff61b7c/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "name":"create_asset_permission_update",
            "user_groups":["745980b1-54be-4c5e-b6ab-89826c2e2054"],
            "nodes":["3728f004-99a2-4fca-9577-84d5ffcf9eff"],
            "accounts":["@ALL"],
            "actions":["connect","upload","download","copy","paste"],
            "is_active":true,
            "date_start":"2025-01-01T00:00:00.879Z",
            "date_expired":"2095-01-01T00:00:00.879Z"
        }'
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
NODE_ID     = "your node id"
PER_ID      = "your permission id"

def update_asset_permissions():
    url = f"{API_URL}/api/v1/perms/asset-permissions/{PER_ID}/"
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
        "name":"create_asset_permission_update",
        "user_groups":[GROUP_ID],
        "nodes":[NODE_ID],
        "accounts":["@ALL"],
        "actions":["connect","upload","download","copy","paste"],
        "is_active": True,
        "date_start":"2025-01-01T00:00:00.879Z",
        "date_expired":"2095-01-01T00:00:00.879Z"
    }

    try:
        response = requests.put(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Asset authorization updated successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    update_asset_permissions()
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| users | Type: String[], user |    |
| user_groups | Type: String[], user group |    |
| assets | Type: String[], asset |    |
| nodes | Type: String[], node |    |
| accounts | Type: String[], authorized account |    |
| actions | Type: String[], action |    |
| is_active | Type: Boolean, activated |    |
| created_by | Type: String, Creator |    |
| comment | Type: String, remarks |    |
| date_created | Type: String(date-time), authorization rule creation time |    |
| date_start | Type: String(date-time), authorization rule start time |    |
| date_expired | Type: String(date-time), authorization rule expiration time |    |

- **Use Case:**

Scenario: The authorization of the database operations team (dba-team) for the production MySQL node is about to expire. The administrator will renew it for one year and temporarily add file upload and download actions in conjunction with this week's release window.

```sh
curl -X PUT 'https://localhost/api/v1/perms/asset-permissions/6e5d4c3b-2a19-4f08-97e6-d5c4b3a29180/' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer <token>' \
-H 'X-JMS-ORG: <organizationID>' \
-d '{
        "name": "dba-team-mysql-prod",
        "user_groups": ["9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"],
        "nodes": ["2f3e4d5c-6b7a-4c8d-9e0f-1a2b3c4d5e6f"],
        "accounts": ["@ALL"],
        "actions": ["connect", "upload", "download"],
        "is_active": true,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2027-07-22T00:00:00.000Z",
        "comment": "Authorization renewal for one year，The release window is temporarily open for file upload and download"
    }'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)
