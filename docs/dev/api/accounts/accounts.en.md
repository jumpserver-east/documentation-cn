## /api/v1/accounts/accounts/

### POST
- **Description:** 
Create asset account

- **Request headers:**  

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type      | `application/json`                            | The request/response body is in JSON format           |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name | Type: String, name | - |
| username | Type: String, username | - |
| secret_type | Type: String, ciphertext type, optional values are `password` (password), `ssh_key` (SSH key), the default is `password` | password |
| secret | Type: String, key/password, only enabled when `password` is selected in the `secret_type` field | - |
| passphrase | Type: String, key password, only enabled when `ssh_key` is selected in the `secret_type` field | - |
| comment | Type: String, remarks | - |
| asset* | Type: String, asset ID (UUID); this field in the response is an AccountAsset object (including id, name, address, etc.), just pass the asset ID string when requesting | - |
| privileged | Type: Boolean, privileged account | - |
| push_now | Type: Boolean, push immediately | false |
| is_active | Type: Boolean, activated | - |

> Note: Parameters with * are required.
- **Return parameters:**  

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| username | Type: String, username |    |
| secret_type | Type: Object (including value and label fields), ciphertext type |    |
| created_by | Type: String, Creator |    |
| comment | Type: String, remarks |    |
| su_from | Type: Object (including id, name, username fields), switch from account |    |
| asset | Type: Object (AccountAsset object, including fields such as id, name, address, type, category, platform, etc.), asset information |    |
| source | Type: Object (including value and label fields), source |    |
| connectivity | Type: Object (including value and label fields), connectivity |    |
| org_id | Type: String, organization |    |
| org_name | Type: String, organization name |    |
| privileged | Type: Boolean, whether there is a privileged account |    |
| is_active | Type: Boolean, activated |    |

**Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/accounts/accounts/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "username": "root",
        "secret_type": "password",
        "asset": "09e1e072-1498-42f7-a6b1-567c2db56f59"
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

def create_assets_accounts():
    url = f"{API_URL}/api/v1/accounts/accounts/"
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
        "username": "root",
        "secret_type": "password",
        "asset": ASSET_ID
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Asset account created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_assets_accounts()
```

- **Use Case:**

Scenario: After the new Linux server `web-server-01` goes online, its deployment account `deploy` is brought under centralized platform management and pushed to the asset immediately after creation, avoiding manual login and password changes.

```sh
curl -X POST 'https://localhost/api/v1/accounts/accounts/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "web-server-01-deploy",
        "username": "deploy",
        "secret_type": "password",
        "secret": "Dep@2026#Init",
        "asset": "09e1e072-1498-42f7-a6b1-567c2db56f59",
        "privileged": false,
        "push_now": true,
        "comment": "web-server-01 Online management，The account is hosted by the platform"
    }'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

### GET
- **Description:**
Check account

- **Request headers:**  

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
|

- **Return parameters:**  

| Field name | Field description | Remarks |
| --- | --- | --- |
| asset | Type: Object, Asset |    |
| connectivity | Type: Object (including value and label fields), connectability |    |
| id | Type: String, id |    |
| name | Type: String, name |    |
| privileged | Type: Boolean, whether it is a privileged account |    |
| username | Type: String, account name |    |
| secret_type | Type: Object, ciphertext type |    |
| source | Type: Object (including value and label fields), source |    |
| is_active | Type: Boolean, activated |    |
| created_by | Type: String, Creator |    |
| date_created | Type: String(date-time), creation time |    |

**Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/accounts/accounts/?asset_id=a014d307-7c2b-4788-a3e0-aebd01ddf761&has_secret=true&offset=0&limit=15' \
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

def search_assets_accounts():
    url = f"{API_URL}/api/v1/accounts/accounts/"
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
        "asset_id": ASSET_ID
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        accounts_data = response.json()
        results = accounts_data.get("results", [])
        total = accounts_data.get("count", 0)
        if total == 0:
            print("No matching asset account found")
        else:
            print(f"Found {total} matching asset account：")
            print(json.dumps(results, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    search_assets_accounts()
```

- **Use Case:**

Scenario: During the quarterly security audit, take inventory of all `root` privileged accounts that are still active in the entire organization, and output a list for auditors to check whether there is unauthorized custody.

```sh
curl -X GET 'https://localhost/api/v1/accounts/accounts/?username=root&privileged=true&is_active=true&limit=100' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

## /api/v1/accounts/accounts/{id}/

### DELETE
- **Description:**
Delete asset account

- **Request headers:**  

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type      | `application/json`                            | The request/response body is in JSON format           |

- **Path Params:**  

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string, asset account | Yes |

**Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/accounts/accounts/44e2c39a-c022-455d-b9b1-889e52e453b7/' \
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
ACCOUNT_ID  = "your account id"

def delete_assets_accounts():
    url = f"{API_URL}/api/v1/accounts/accounts/{ACCOUNT_ID}/"
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
        print("Asset account deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_assets_accounts()
```

- **Use Case:**

Scenario: The database server `db-server-02` has been offline as planned. The operations team cleans up the remaining managed account `dba_backup` during the asset withdrawal process to prevent invalid credentials from remaining on the platform.

```sh
curl -X DELETE 'https://localhost/api/v1/accounts/accounts/6f2ab8c1-3d54-4e0a-9c77-1b2f0c5d8e9a/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

### PUT / PATCH
- **Description:**
Update asset account

- **Request headers:**  

| Key (Header)      | Example value                                       | Description                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type      | `application/json`                            | The request/response body is in JSON format           |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name | Type: String, name | - |
| username | Type: String, username | - |
| secret_type | Type: String, ciphertext type, optional values are `password` (password), `ssh_key` (SSH key), the default is `password` | password |
| secret | Type: String, key/password, only enabled when `password` is selected in the `secret_type` field | - |
| passphrase | Type: String, key password, only enabled when `ssh_key` is selected in the `secret_type` field | - |
| comment | Type: String, remarks | - |
| asset* | Type: String, asset ID (UUID); this field in the response is an AccountAsset object (including id, name, address, etc.), just pass the asset ID string when requesting | - |
| privileged | Type: Boolean, privileged account | - |
| push_now | Type: Boolean, push immediately | false |
| is_active | Type: Boolean, activated | - |

> Note: Parameters with * are required (required constraints only apply to PUT; all fields are optional when PATCH).
- **Return parameters:**  

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, id |    |
| name | Type: String, name |    |
| username | Type: String, username |    |
| secret_type | Type: Object (including value and label fields), ciphertext type |    |
| created_by | Type: String, Creator |    |
| comment | Type: String, remarks |    |
| su_from | Type: Object (including id, name, username fields), switch from account |    |
| asset | Type: Object (AccountAsset object, including fields such as id, name, address, type, category, platform, etc.), asset information |    |
| source | Type: Object (including value and label fields), source |    |
| connectivity | Type: Object (including value and label fields), connectivity |    |
| org_id | Type: String, organization |    |
| org_name | Type: String, organization name |    |
| privileged | Type: Boolean, whether there is a privileged account |    |
| is_active | Type: Boolean, activated |    |

**Request Example**

**CURL**
```sh
curl -X PUT 'https://localhost/api/v1/accounts/accounts/f3280232-113a-4135-a908-eddd1d9f27b6/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "username": "root_update",
        "secret_type": "password",
        "asset": "bf13858e-b43a-45d1-b952-5624869fdbce"
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
ACCOUNT_ID  = "your account id"

def update_assets_accounts():
    url = f"{API_URL}/api/v1/accounts/accounts/{ACCOUNT_ID}/"
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
        "username": "root",
        "secret_type": "password",
        "asset": ASSET_ID
    }

    try:
        response = requests.put(
            url, auth = auth, headers = headers,
            data=json.dumps(data)
        )
        response.raise_for_status()
        print("Account updated successfully:")
        print(json.dumps(response.json(), indent=2))
    except Exception as e:
        print(f"Error:{e}")
        print(f"Response content:{response.text if 'response' in locals() else 'No response'}")

if __name__ == "__main__":
    update_assets_accounts()
```

- **Use Case:**

Scenario: During the emergency response, it was discovered that the account `appadmin` was suspected to be leaked. After the operations manually reset the password on the target host, PATCH was used to synchronize only the ciphertext hosted by the platform and add comments, without changing other attributes of the account.

```sh
curl -X PATCH 'https://localhost/api/v1/accounts/accounts/f3280232-113a-4135-a908-eddd1d9f27b6/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "secret_type": "password",
        "secret": "Emg@Reset#0722",
        "comment": "2026-07-22 Emergency password change，work order INC-20260722-013"
    }'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)
