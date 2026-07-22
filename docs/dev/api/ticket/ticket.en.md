## /api/v1/tickets/apply-asset-tickets/open/

### POST

- **Description:**
Create a work order

- **Request headers:**  

| key              | value                           | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator's token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | Output in json format              

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| title* | Type: String, work order title | - |
| org_id* | Type: String, organization | - |
| apply_nodes | Type: Object[] (object array, each element contains id and name fields), requested nodes | Supports fuzzy search and displays up to 10 items |
| apply_assets | Type: string[], application asset id | Support fuzzy search, display up to 10 items |
| apply_accounts | Type: String[], application account id | "@ALL": all accounts; "@SPEC": specified account; "@INPUT": manual account; "@USER": account with the same name |
| apply_actions | Type: String[], action | Default: []; optional values: [connect, upload, download, copy, paste, delete, share] |
| apply_date_start | Type: String(datetime), start date | - |
| apply_date_expired | Type: String (datetime), expiration date (the original text "Expiration Log" should be a clerical error) | - |
| comment | Type: String, remarks | - |
> Note: Parameters with * are required.


- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| title | Type: String, title |    |
| org_id | Type: String, organization |    |
| comment | Type: String, remarks |    |
| type | Type: String, type |    |
| apply_nodes | Type: String[], application node |    |
| apply_assets | Type: String[], apply for assets |    |
| apply_accounts | Type: String[], apply for an account |    |
| apply_actions | Type: String[], application action |    |
| serial_num | Type: String, serial number |    |
| approval_step | Type: String, process step |    |
| state | Type: String, work order action |    |
| status | Type: String, work order status |    |
| applicant | Type: String, applicant |    |
| org_name | Type: String, organization name |    |
| apply_permission_name | Type: String, work order authorization name |    |
| apply_date_start | Type: String(date-time), application start time |    |
| apply_date_expired | Type: String(date-time), application end time |    |


- **Request Example:**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/tickets/apply-asset-tickets/open/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title":"test_tickets_1",
        "apply_accounts":["@ALL"],
        "apply_actions":["connect"],
        "org_id":"00000000-0000-0000-0000-000000000002",
        "apply_assets":["b4f205af-4353-49ef-befa-ff9095d52a27"],
        "apply_date_start":"2023-03-28T02:10:23.245Z",
        "apply_date_expired":"2023-04-04T02:10:23.245Z"
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

def apply_asset_tickets():
    url = f"{API_URL}/api/v1/tickets/apply-asset-tickets/open/"
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
        "title":"test_tickets_1",
        "apply_accounts":["@ALL"],
        "apply_actions":["connect"],
        "org_id": ORG_ID,
        "apply_date_start":"2025-01-01T00:00:00.245Z",
        "apply_date_expired":"2095-01-01T00:00:00.245Z"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Work order created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    apply_asset_tickets()
```

- **Use Case:**

Scenario: After the external ITSM system approves the project, it automatically initiates an asset access application in JumpServer for the database administrator zhangsan, and applies to connect to the production database mysql-prod-01 with the account of the same name. The authorization is valid for one week.

```sh
curl -X POST 'https://localhost/api/v1/tickets/apply-asset-tickets/open/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "title": "zhangsan Request access to production database mysql-prod-01",
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["c3d5f1a2-7b8e-4f6d-9a0c-1e2f3a4b5c6d"],
        "apply_accounts": ["@USER"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T09:00:00.000Z",
        "apply_date_expired": "2026-07-29T09:00:00.000Z",
        "comment": "ITSM work order INC-20260722-001 Related application"
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: Interfacing with External Work Order Systems](../examples/external_ticket.md)

## /api/v1/tickets/tickets/

### GET
- **Description:**
Get a work order

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator's token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type    | application/json                        | Output in json format                                                       |

- **Query parameters (Query):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| state | Type: String, action | - |
|    | Optional values: pending, approved, rejected |    |
| status | Type: String, status | - |
|    | Optional values: closed (closed, rejected), open (open) |    |
| type | Type: String, type | - |
|    | Optional values: apply_asset (apply for assets), login_confirm (user login review), command_confirm (command review), login_asset_confirm (asset login review) |    |

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| title | Type: String, title |    |
| org_id | Type: String, organization |    |
| serial_num | Type: String, number |    |
| approval_step | Type: String, work order approval step |    |
| type | Type: String, type |    |
| state | Type: String, action |    |
| applicant | Type: String, applicant |    |
| status | Type: String, status |    |
| org_name | Type: String, organization name |    |
| date_created | Type: String(date-time), creation time |    |
| date_updated | Type: string(date-time), update time |    |


- **Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/?state=pending&status=open&type=apply_asset' \
    -H 'Content-Type: application/json' \
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

def search_tickets():
    url = f"{API_URL}/api/v1/tickets/tickets/"
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
        "state": "pending",
        "status": "open",
        "type": "apply_asset"
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        nodes_data = response.json()
        if nodes_data.get("count", 0) == 0:
            print(f"Work order not found")
        else:
            print(f"Found {nodes_data['count']} matching work orders：")
            print(json.dumps(nodes_data, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    search_tickets()
```

- **Use Case:**

Scenario: The duty inspection script polls all open and pending asset login review work orders every hour. If a backlog is found, the duty administrator is reminded to handle it in a timely manner through corporate WeChat.

```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/?state=pending&status=open&type=login_asset_confirm' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: Interfacing with External Work Order Systems](../examples/external_ticket.md)

## /api/v1/tickets/apply-asset-tickets/{id}/approve/

### PATCH
- **Description:**
Approval work order

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator's token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type    | application/json                        | Output in json format                                                       |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| org_id | Type: String, organization ID | - |
| apply_nodes | Type: String[], apply for node id | - |
| apply_assets | Type: string[], application asset id | - |
| apply_accounts | Type: String[], application account id | "@ALL": all accounts; "@SPEC": specified account; "@INPUT": manual account; "@USER": account with the same name |
| apply_actions | Type: String[], action | Default: []; optional values: [connect, upload, download, copy, paste, delete, share] |
| apply_date_start | Type: String(datetime), start date (the original format "String(date time)" is corrected to the standard datetime format expression) | - |
| apply_date_expired | Type: String(datetime), expiration date (the original text "Expiration Log" should be a clerical error, the format "String(date time)" is corrected to the standard datetime format expression) | - |
| comment | Type: String, remarks | - |

- **Request Example**

**CURL**
```sh
curl -X PATCH 'https://localhost/api/v1/tickets/apply-asset-tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/approve/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["b4f205af-4353-49ef-befa-ff9095d52a27"],
        "apply_accounts": ["@ALL"],
        "apply_actions": ["connect"],
        "apply_date_start": "2025-03-28 00:00:00",
        "apply_date_expired": "2025-04-04 00:00:00"
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
TICKET_ID   = "your ticket id"
ASSET_ID    = "your asset id"

def approve_tickets():
    url = f"{API_URL}/api/v1/tickets/apply-asset-tickets/{TICKET_ID}/approve/"
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
            "org_id": ORG_ID,
            "apply_assets": [ASSET_ID],
            "apply_accounts": ["@ALL"],
            "apply_actions": ["connect"],
            "apply_date_start": "2025-03-28 00:00:00",
            "apply_date_expired": "2025-04-04 00:00:00"
    }

    try:
        response = requests.patch(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Work order approved successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    approve_tickets()
```

- **Use Case:**

Scenario: After the manager in the external work order system passes the approval, the JumpServer is called back to automatically release the corresponding asset application work order, granting only the connection and file download permissions to web-server-01, and tightening the authorization until it expires before get off work this Friday.

```sh
curl -X PATCH 'https://localhost/api/v1/tickets/apply-asset-tickets/9f8e7d6c-5b4a-3210-fedc-ba9876543210/approve/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
        "apply_accounts": ["@SPEC"],
        "apply_actions": ["connect", "download"],
        "apply_date_start": "2026-07-22 09:00:00",
        "apply_date_expired": "2026-07-24 18:00:00",
        "comment": "External work order INC-20260722-001 Approval passed，automatic release"
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: Interfacing with External Work Order Systems](../examples/external_ticket.md)

## /api/v1/tickets/flows/

### GET
- **Description:**
Query process

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator's token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | Output in json format      

- **Query parameters (Query):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: Int, paging offset | - |

- **Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/tickets/flows/?offset=0&limit=15' \
    -H 'Content-Type: application/json' \
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

def search_flows():
    url = f"{API_URL}/api/v1/tickets/flows/"
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
        nodes_data = response.json()
        if nodes_data.get("count", 0) == 0:
            print(f"Process not found")
        else:
            print(f"Found {nodes_data['count']} a matching process：")
            print(json.dumps(nodes_data, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    search_flows()
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| type | Type: Object, process type |    |
| org_id | Type: String, organization |    |
| org_name | Type: String, organization name |    |
| approval_level | Type: int, approval level |    |
| rules | Type: Array, approval process | The element is a TicketFlowApprove object, containing two fields: level (type: int, approval level, read-only) and users (approval user) |
| created_by | Type: String, creator |    |
| date_created | Type: String(date-time), creation time |    |
| date_updated | Type: string(date-time), update time |    |

- **Use Case:**

Scenario: In the preparation stage before connecting to the external work order system, the integrated script pulls the approval process configured by the current organization in pages, and checks whether the approval levels and approvers of various work orders comply with the company's security regulations.

```sh
curl -X GET 'https://localhost/api/v1/tickets/flows/?offset=0&limit=10' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: Interfacing with External Work Order Systems](../examples/external_ticket.md)

## /api/v1/tickets/flows/{id}/
### PATCH
- **Description:**
Update process

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator's token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | Output in json format                           

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| type | Type: String, type | - |
| approval_level | Type: int, approval level | - |
| rules | Type: Array, the element is a TicketFlowApprove object (including level: int, approval level, readOnly only returns the response; users: approver, the only writable field) | - |

**Request Example**

**CURL**
```sh
curl -X PATCH 'https://localhost/api/v1/tickets/flows/41b36621-dd4d-492e-a72c-be20b2daeea8/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "type": "apply_asset",
        "approval_level": 1,
        "rules": [{
            "users": []
        }]
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
TICKET_ID   = "your ticket id"

def update_tickets_flows():
    url = f"{API_URL}/api/v1/tickets/flows/{TICKET_ID}/"
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
        "type": "apply_asset",
        "approval_level": 1,
        "rules": [{
            "users": []
        }]
    }

    try:
        response = requests.patch(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Modification process successful:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    update_tickets_flows()
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| type | Type: Object, process type |    |
| org_id | Type: String, organization |    |
| org_name | Type: String, organization name |    |
| approval_level | Type: int, approval level |    |
| rules | Type: Array, approval process |    |
| level | Type: int, process level |    |
| strategy | Type: object, approval role |    |
| assignees_display | Type: Array, approver name |    |
| created_by | Type: String, creator |    |
| date_created | Type: String(date-time), creation time |    |
| date_updated | Type: string(date-time), update time |    |

- **Use Case:**

Scenario: Security rectification requires asset application work orders to be approved at two levels. The operations platform adjusts the approval level of this process to level 2 and designates the security group approver lisi as the second-level approver.

```sh
curl -X PATCH 'https://localhost/api/v1/tickets/flows/6d5c4b3a-2f1e-0d9c-8b7a-654321fedcba/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "type": "apply_asset",
        "approval_level": 2,
        "rules": [
            {"users": ["f0e1d2c3-b4a5-6789-0123-456789abcdef"]},
            {"users": ["1a2b3c4d-5e6f-7a8b-9c0d-e1f2a3b4c5d6"]}
        ]
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: Interfacing with External Work Order Systems](../examples/external_ticket.md)
