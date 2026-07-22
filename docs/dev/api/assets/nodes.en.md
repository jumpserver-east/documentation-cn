## /api/v1/assets/nodes/

### GET
- **Description:**
Query node

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| search | Type: String, search term, supports node name search | - |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, node id |    |
| key | Type: String, key |    |
| value | Type: String, value\node name |    |
| org_id | Type: String, organization |    |
| name | Type: String, node name (read-only) |    |
| full_value | Type: String, full name |    |
| org_name | Type: String, organization name |    |

**Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/?search=/Default/xxcompany' \
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
SEARCH_WORD = "your search word"

def search_nodes(keyword):
    url = f"{API_URL}/api/v1/assets/nodes/"
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

    params = {"search": keyword}

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        nodes_data = response.json()
        nodes = nodes_data.get("results", [])
        if not nodes:
            print("No matching asset node found")
        else:
            print(f"Found {nodes_data['count']} matching asset nodes：")
            print(json.dumps(nodes_data, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    search_nodes(SEARCH_WORD)
```
- **Response example:**

```json
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": "aa2aa3fe-c2c3-49ca-b4bd-d04dc8bf6693",
      "key": "1",
      "value": "DEFAULT",
      "org_id": "00000000-0000-0000-0000-000000000002",
      "name": "DEFAULT",
      "full_value": "/DEFAULT",
      "org_name": "DEFAULT"
    }
  ]
}
```

- **Use Case:**

Scenario: The CMDB system performs asset structure reconciliation with the bastion host. First, search for the "production environment" node by node name to confirm whether it already exists, and limit the return of 20 items per page for paging traversal.

```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/?search=production environment&limit=20&offset=0' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)

##  /api/v1/assets/nodes/{id}/children/

### POST
- **Description:**
Create node

- **Request headers:**

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type      | `application/json`                            | The request/response body is in JSON format           |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: String, node id | Yes |

- **Request body parameters (Body):**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| value | Type: String, node name; this field is not required (nullable) in the swagger definition, but it is recommended to pass it in to specify the node name when creating a node. | - |

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, node id |    |
| key | Type: String, key |    |
| value | Type: String, value\node name |    |
| org_id | Type: String, organization |    |
| name | Type: String, node name (read-only) |    |
| full_value | Type: String, full name |    |
| org_name | Type: String, organization name |    |

**Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/assets/nodes/3728f004-99a2-4fca-9577-84d5ffcf9eff/children/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{"value":"test_create_node"}'
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
NODE_NAME   = "your node name"

def create_children_nodes(node_id, node_name):
    url = f"{API_URL}/api/v1/assets/nodes/{node_id}/children/"
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
        "value": node_name
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Asset node created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_children_nodes(NODE_ID, NODE_NAME)
```

- **Use Case:**

Scenario: The new business line "e-commerce middle platform" is online, and the operations platform automatically creates a sub-node with the same name under the "production environment" node (ID is `3728f004-99a2-4fca-9577-84d5ffcf9eff`) for subsequent batch management of servers of this business line.

```sh
curl -X POST 'https://localhost/api/v1/assets/nodes/3728f004-99a2-4fca-9577-84d5ffcf9eff/children/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{"value":"E-commerce middle platform"}'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)

##  /api/v1/assets/nodes/{id}/ 

### DELETE
- **Description:**
Delete node

- **Request headers:**

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type      | `application/json`                            | The request/response body is in JSON format           |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: String, node id | Yes |


**Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/assets/nodes/89c68ef6-7790-4f20-8f8d-fdd76d229b3d/' \
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
NODE_ID     = "your node id"

def delete_assets_nodes(node_id):
    url = f"{API_URL}/api/v1/assets/nodes/{node_id}/"
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
        print("Asset node deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_assets_nodes(NODE_ID)
```

- **Use Case:**

Scenario: The old project "Reporting System" is offline as a whole, and all its assets have been moved out. The administrator deletes the empty node corresponding to the project (ID is `89c68ef6-7790-4f20-8f8d-fdd76d229b3d`) in the cleanup script to keep the asset tree clean.

```sh
curl -X DELETE 'https://localhost/api/v1/assets/nodes/89c68ef6-7790-4f20-8f8d-fdd76d229b3d/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical case: external system applies for assets and automatically authorizes them](../examples/asset_sync_authorize.md)
