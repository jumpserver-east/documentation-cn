> **Note:** The password change plans and execution endpoints (`/api/v1/accounts/change-secret-automations/`, `/api/v1/accounts/change-secret-automations/{id}/`, `/api/v1/accounts/change-secret-executions/`) described on this page are JumpServer Enterprise Edition (XPack) features and are not included in this repository's `swagger.yml` (Community Edition) baseline. Refer to the JumpServer Enterprise Edition API schema for field definitions.

## /api/v1/accounts/change-secret-automations/

### GET
- **Description:**
Query password change plan

- **Request headers:** 

| key | value | Remarks |
|----|----|------|
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | b96810faac725563304dada8c323c4fa061863d4 is the administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | Output in json format |

**Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/accounts/change-secret-automations/?offset=0&limit=15' \
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

def search_change_secret_automations(keyword):
    url = f"{API_URL}/api/v1/accounts/change-secret-automations/"
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
        "search": keyword
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        nodes_data = response.json()
        count = nodes_data.get("count", 0)
        if count == 0:
            print(f"No password change plan found")
        else:
            print(f"Found {count} matching password change plan(s):")
            print(json.dumps(nodes_data.get("results", []), indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    search_change_secret_automations(SEARCH_WORD)
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| count | Type: Int, total |    |
| next | Type: String, next page link |    |
| previous | Type: String, previous page link |    |
| results | Type: List, data |    |
| id | Type: String, id |    |
| name | Type: String, name |    |
| accounts | Type: String[], asset account name |    |
| assets | Type: List[Object], asset ID |    |
| nodes | Type: List[Object], asset node |    |
| is_active | Type: Boolean, whether to activate |    |
| is_periodic | Type: Boolean, whether to execute regularly |    |
| crontab | Type: String, execute crontab expression regularly |    |
| interval | Type: String, periodic execution |    |
| secret_strategy | Type: String, ciphertext generation strategy |    |
| secret_type | Type: Object, ciphertext type |    |
| secret | Type: String, password |    |
| password_rules | Type: Object, random password length |    |
| recipients | Type: List[Object], recipient |    |
| org_id | Type: String, organization ID |    |
| org_name | Type: String, organization name |    |
| comment | Type: String, remarks |    |
| date_created | Type: String[date], creation time |    |
| date_updated | Type: String[date], update time |    |
| created_by | Type: String, creator |    |

- **Use Case:**

Scenario: Before the quarterly security audit, operations personnel search by name for password change plans related to the production environment and confirm that production asset accounts are covered by scheduled password rotation.

```sh
curl -X GET 'https://localhost/api/v1/accounts/change-secret-automations/?search=prod&offset=0&limit=15' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)


### POST
- **Description:**
Create a password change plan

- **Request headers:** 

| key | value | Remarks |
|----|----|------|
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | b96810faac725563304dada8c323c4fa061863d4 is the administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | Output in json format |

**Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/accounts/change-secret-automations/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "accounts": ["root"],
        "secret_strategy": "random",
        "secret_type": "password",
        "password_rules": {
            "length": "16"
        },
        "is_periodic": true,
        "interval": 24,
        "is_active": true,
        "name": "test",
        "assets": ["9266b1f8-f74d-482c-805a-6eed0e099a42"],
        "comment": "test"
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
ASSET_ID    = "your asset id"

def create_change_secret_automations():
    url = f"{API_URL}/api/v1/accounts/change-secret-automations/"
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
        "accounts": ["root"],
        "secret_strategy": "random",
        "secret_type": "password",
        "password_rules": {
            "length": "16"
        },
        "is_periodic": True,
        "interval": 24,
        "is_active": True,
        "name": "test",
        "assets": [ASSET_ID],
        "comment": "test"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Password change plan created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_change_secret_automations()
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| accounts | Type: String[], asset account name |    |
| assets | Type: List[Object], asset ID |    |
| nodes | Type: List[Object], asset node |    |
| is_active | Type: Boolean, whether to activate |    |
| is_periodic | Type: Boolean, whether to execute regularly |    |
| crontab | Type: String, execute crontab expression regularly |    |
| interval | Type: String, periodic execution |    |
| secret_strategy | Type: String, ciphertext generation strategy |    |
| secret_type | Type: Object, ciphertext type |    |
| secret | Type: String, password |    |
| password_rules | Type: Object, random password length |    |
| recipients | Type: List[Object], recipient |    |
| org_id | Type: String, organization ID |    |
| org_name | Type: String, organization name |    |
| comment | Type: String, remarks |    |
| date_created | Type: String[date], creation time |    |
| date_updated | Type: String[date], update time |    |
| created_by | Type: String, creator |    |

- **Use Case:**

Scenario: A batch of newly delivered MySQL database servers come online, and a random password change plan is created for the root account that automatically rotates at 3 a.m. every Saturday. The password length is 20 characters.

```sh
curl -X POST 'https://localhost/api/v1/accounts/change-secret-automations/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "mysql-root-weekly-rotate",
        "accounts": ["root"],
        "assets": ["1f6a2c3e-8b4d-4f2a-9c1e-5d7b8a9e0f12"],
        "secret_strategy": "random",
        "secret_type": "password",
        "password_rules": {
            "length": "20"
        },
        "is_periodic": true,
        "crontab": "0 3 * * 6",
        "is_active": true,
        "comment": "new delivery MySQL server root Account rotation every week"
    }'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)


## /api/v1/accounts/change-secret-automations/{id}/
### PUT
- **Description:**
Update password change plan

- **Request headers:** 

| key | value | Remarks |
|----|----|------|
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | b96810faac725563304dada8c323c4fa061863d4 is the administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | Output in json format |

- **Path parameter (Path):** 

| Parameter name | Description | Default value |
| --- | --- | --- |
| id* | Type: String, password change plan ID, passed through the URL path | - |

- **Request body parameters (Body):** 

| Parameter name | Description | Default value |
| --- | --- | --- |
| name* | Type: String, name | - |
| accounts* | Type: String[], asset account name | - |
| assets | Type: String[], asset ID | - |
| nodes | Type: string[], asset node ID | - |
| is_active* | Type: Boolean, whether to activate | - |
| is_periodic* | Type: Boolean, whether to execute regularly | - |
| crontab | Type: String, execute crontab expression regularly | Fill in when is_periodic=true |
| interval | Type: String, periodic execution | Fill in when is_periodic=true, default 24 |
| secret_strategy* | Type: String, ciphertext generation strategy | Default: specify specific; random: random |
| secret_type* | Type: String, ciphertext type | Default: password; optional value: ssh_key |
| password_rules | Type: Object, password generation rules. Subfield: length (int, password length, range 8-36, default 16); optional subfield: lowercase/uppercase/digit/symbol (Boolean), exclude_symbols (String) | Default 16; required when secret_strategy=random and secret_type=password |
| secret | Type: String, password | Required when secret_strategy=specific and secret_type=password |
| comment | Type: String, remarks | - |

> Note: Parameters with * are required.
**Request Example**

**CURL**
```sh
curl -X PUT 'https://localhost/api/v1/accounts/change-secret-automations/0a6a2e40-f92b-4aca-94ef-5f6ae5b0966c/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "accounts": ["root"],
        "secret_strategy": "random",
        "secret_type": "password",
        "password_rules": {
            "length": "16"
        },
        "is_periodic": true,
        "interval": 24,
        "is_active": true,
        "name": "test_update",
        "assets": ["9266b1f8-f74d-482c-805a-6eed0e099a42"],
        "comment": "test"
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
ASSET_ID    = "your asset id"
MIS_ID      = "your mission id"

def update_change_secret_automations():
    url = f"{API_URL}/api/v1/accounts/change-secret-automations/{MIS_ID}/"
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
        "accounts": ["root"],
        "secret_strategy": "random",
        "secret_type": "password",
        "password_rules": {
            "length": "16"
        },
        "is_periodic": True,
        "interval": 24,
        "is_active": True,
        "name": "test",
        "assets": [ASSET_ID],
        "comment": "test"
    }

    try:
        response = requests.put(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Password change plan updated successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    update_change_secret_automations()
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| accounts | Type: String[], asset account name |    |
| assets | Type: List[Object], asset ID |    |
| nodes | Type: List[Object], asset node |    |
| is_active | Type: Boolean, whether to activate |    |
| is_periodic | Type: Boolean, whether to execute regularly |    |
| crontab | Type: String, execute crontab expression regularly |    |
| interval | Type: String, periodic execution |    |
| secret_strategy | Type: String, ciphertext generation strategy |    |
| secret_type | Type: Object, ciphertext type |    |
| secret | Type: String, password |    |
| password_rules | Type: Object, random password length |    |
| recipients | Type: List[Object], recipient |    |
| org_id | Type: String, organization ID |    |
| org_name | Type: String, organization name |    |
| comment | Type: String, remarks |    |
| date_created | Type: String[date], creation time |    |
| date_updated | Type: String[date], update time |    |
| created_by | Type: String, creator |    |

- **Use Case:**

Scenario: The company's password security policy is upgraded, the random password length of the existing password change plan is increased from 16 to 24 characters, and the execution cycle is shortened from weekly to every 24 hours.

```sh
curl -X PUT 'https://localhost/api/v1/accounts/change-secret-automations/0a6a2e40-f92b-4aca-94ef-5f6ae5b0966c/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "mysql-root-weekly-rotate",
        "accounts": ["root"],
        "assets": ["1f6a2c3e-8b4d-4f2a-9c1e-5d7b8a9e0f12"],
        "secret_strategy": "random",
        "secret_type": "password",
        "password_rules": {
            "length": "24"
        },
        "is_periodic": true,
        "interval": 24,
        "is_active": true,
        "comment": "Security policy upgrade：Password length 24 Bit，every 24 hour rotation"
    }'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

### DELETE
- **Description:**
Delete password change plan

- **Request headers:**  

| Key (Header) | Example value | Description |
|-------------|--------|------|
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Path parameter (Path):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| id* | Type: String, password change plan ID, passed through the URL path (the DELETE request has no request body) | - |

> Note: Parameters with * are required.
**Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/accounts/change-secret-automations/0a6a2e40-f92b-4aca-94ef-5f6ae5b0966c/' \
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
MIS_ID      = "your mission id"

def delete_change_secret_automations():
    url = f"{API_URL}/api/v1/accounts/change-secret-automations/{MIS_ID}/"
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
        print("Password change plan deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_change_secret_automations()
```

- **Use Case:**

Scenario: A batch of legacy Windows servers has been decommissioned, so their associated password change plans are no longer needed. Operations personnel delete the plans as the final asset cleanup step to prevent unnecessary tasks and errors.

```sh
curl -X DELETE 'https://localhost/api/v1/accounts/change-secret-automations/7c3d9e5a-2b1f-4e8c-a6d4-9f0b3c7e1a58/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

## /api/v1/accounts/change-secret-executions/

### POST
- **Description:**
Execute password change plan

- **Request headers:**  

| Key (Header) | Example value | Description |
|-------------|--------|------|
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| automation* | Type: String, password change plan ID | - |


> Note: Parameters with * are required.
**Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/accounts/change-secret-executions/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "automation": "bc778562-630e-4c89-971a-3ec629d4fd3f"
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
MIS_ID      = "your mission id"

def automations_executions():
    url = f"{API_URL}/api/v1/accounts/change-secret-executions/"
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
        "automation": MIS_ID
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data),
            verify = False
        )
        response.raise_for_status()
        print(f"The password change plan has been executed")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    automations_executions()
```

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| task | Type: String, task id |    |

- **Use Case:**

Scenario: On the day when the core operations personnel leave the company, the security team immediately manually triggers the password change plan for the production servers they have contacted without waiting for the periodic schedule, realizing instant rotation of privileged account passwords.

```sh
curl -X POST 'https://localhost/api/v1/accounts/change-secret-executions/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "automation": "bc778562-630e-4c89-971a-3ec629d4fd3f"
    }'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)
