## /api/v1/assets/hosts/

### POST
- **Description:**
Create a host asset

- **Request headers:**  

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type      | `application/json`                            | The request/response body is in JSON format           |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name* | Type: String, name | - |
| address* | Type: String, IP address | - |
| platform* | Type: Object, system platform, platform object containing id/name/type attributes, parameter format example: `{"id": 1}`, platform ID example value: 1 (representing Linux), 5 (representing Windows) | - |
| nodes | Type: String[], node | [] |
| protocols | Type: Object[], protocol/port, parameter format example: `{"name": "ssh", "port": 22}` | - |
| labels | Type: String[], label | - |
| is_active | Type: Boolean, activation status | true |
| accounts | Type: String[], account information | - |


> Note: Parameters with * are required.
- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| address | Type: String, ip |    |
| comment | Type: String, remarks |    |
| platform | Type: String, system platform |    |
| nodes | Type: String[], node |    |
| labels | Type: String[], label |    |
| protocols | Type: Object[], protocol/port, element format example: `{"name": "ssh", "port": 22}` |    |
| nodes_display | Type: String[], asset node name |    |
| category | Type: String, Category |    |
| type | Type: String, type |    |
| is_active | Type: Boolean, activation status |    |
| date_created | Type: String(date-time), creation time |    |
| org_id | Type: String, organization |    |
| org_name | Type: String, organization name |    |


- **Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/assets/hosts/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name":"test_create_asset",
        "address":"192.168.1.1",
        "platform": {"id":1},
        "nodes": [
            {"id":"1ecb988f-ded3-4b57-bc8f-808467abbe2f"}
        ]
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
NODE_ID     = "your node id"

def create_assets():
    url = f"{API_URL}/api/v1/assets/hosts/"
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
        "name":"test_create_asset",
        "address":"192.168.1.1",
        "platform": {"id":1},
        "nodes": [
            {"id": NODE_ID}
        ]
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Asset created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_assets()
```

- **Use Case:**

Scenario: A new production environment Linux application server is added to the CMDB system. The operations platform automatically manages it to the production node of JumpServer and registers the non-standard SSH port according to company specifications.

```sh
curl -X POST 'https://localhost/api/v1/assets/hosts/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "web-server-01",
        "address": "10.10.20.11",
        "platform": {"id": 1},
        "protocols": [
            {"name": "ssh", "port": 22022}
        ],
        "nodes": [
            {"id": "1ecb988f-ded3-4b57-bc8f-808467abbe2f"}
        ],
        "is_active": true
    }'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)



## /api/v1/assets/hosts/{id}/
### DELETE
- Description:
Delete host asset interface

- **Request headers:**  

| key              | value                           | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator's token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type    | application/json                        | Output in json format               |


- **Path Params:**  

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string, asset ID | Yes |

- **Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/assets/hosts/1b56ff93-18e9-478e-97c2-8b6a3720b95d/' \
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
ASSET_ID    = "your asset id"

def delete_assets_hosts():
    url = f"{API_URL}/api/v1/assets/hosts/{ASSET_ID}/"
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
        print("Asset deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_assets_hosts()
```

- **Use Case:**

Scenario: An old database server is decommissioned and offline after completing business migration. The asset recovery process calls the interface to delete it from the JumpServer to avoid remaining invalid assets and authorization entries.

```sh
curl -X DELETE 'https://localhost/api/v1/assets/hosts/3f8c9a52-7d14-4e06-b2ab-56cd7e10f983/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)
