## /api/v1/accounts/account-backup-plans/

### GET
- **Description:**
Query account backup strategy

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | b96810faac725563304dada8c323c4fa061863d4 is the administrator’s token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type    | application/json                        | Output in json format                                                       |

- **Query parameters (Query):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name | Type: String, name | - |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: Int, paging offset | - |

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| count | Type: Int, total |    |
| next | Type: String, next page link |    |
| previous | Type: String, previous page link |    |
| results | Type: List, user data |    |
| id | Type: String, backup policy ID |    |
| name | Type: String, name |    |
| org_id | Type: String, organization ID |    |
| org_name | Type: String, organization name |    |
| is_periodic | Type: Boolean, whether to execute in time |    |
| interval | Type: Int, periodic execution |    |
| crontab | Type: String, execute Crontab expression regularly |    |
| recipients_part_one | Type: List[Object], recipient (first part), item attribute id/name |    |
| recipients_part_two | Type: List[Object], recipient (second part), item attribute id/name |    |
| obj_recipients_part_one | Type: List[Object], object storage recipient (first part), item attribute id/name |    |
| obj_recipients_part_two | Type: List[Object], object storage recipient (second part), item attribute id/name |    |
| types | Type: String[], asset type | linux, windows, unix, other, general, switch, "router", "firewall", "mysql", "mariadb", "postgresql", "oracle", "sqlserver", "clickhouse", "mongodb", "redis", "public", "private", "k8s", "website" |
| date_created | Type: String[date], creation time |    |
| date_updated | Type: String[date], update time |    |
| created_by | Type: String, creator |    |
| comment | Type: String, remarks |    |

- **Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/accounts/account-backup-plans/?offset=0&limit=15' \
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

def search_account_backup_plans(keyword):
    url = f"{API_URL}/api/v1/accounts/account-backup-plans/"
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
        if nodes_data["count"] == 0:
            print("Account backup policy not found")
        else:
            print(f"Found {nodes_data['count']} matching account backup strategies：")
            print(json.dumps(nodes_data["results"], indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    search_account_backup_plans(SEARCH_WORD)
```

- **Use Case:**

Scenario: Before the quarterly security audit, the auditor needs to confirm whether the backup policy named "Production Database Account Backup" is still being executed regularly, and filter the detailed configuration of the policy by name.

```sh
curl -X GET 'https://localhost/api/v1/accounts/account-backup-plans/?name=Production database account backup&offset=0&limit=10' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```




### POST
- **Description:**
Create an account backup strategy

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | b96810faac725563304dada8c323c4fa061863d4 is the administrator’s token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type    | application/json                        | Output in json format                                                       |

- **Request body parameters (Body):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| name* | Type: String, name, maxLength 128 | - |
| types | Type: String[], asset type | [] |
| is_periodic | Type: Boolean, whether to execute in time | true |
| interval | Type: Int, periodic execution interval (1-65535), optional | 24 |
| crontab | Type: String, execute Crontab expression regularly, maxLength 128, can be empty | - |
| comment | Type: String, remarks | - |
| recipients_part_one | Type: List[Object], recipient (first part), item attribute id/name | - |
| recipients_part_two | Type: List[Object], recipient (second part), item attribute id/name | - |
| accounts | Type: List, Account | [] |
| nodes | Type: List[Object], node, item attribute id/name | - |
| assets | Type: List[Object], asset, item attribute id/name | - |
| backup_type | Type: Object, backup type (value/label) | email |
| obj_recipients_part_one | Type: List[Object], object storage recipient (first part), item attribute id/name | - |
| obj_recipients_part_two | Type: List[Object], object storage recipient (second part), item attribute id/name | - |
| zip_encrypt_password | Type: String, compressed file encryption password (writeOnly), optional | - |
| is_active | Type: Boolean, whether to activate | true |
| is_password_divided_by_email | Type: Boolean, whether to split the email password | true |
| is_password_divided_by_obj_storage | Type: Boolean, whether the object storage password is split | true |

> Note: Parameters with * are required.
- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String, backup policy ID |    |
| name | Type: String, name |    |
| org_id | Type: String, organization ID |    |
| org_name | Type: String, organization name |    |
| is_periodic | Type: Boolean, whether to execute in time |    |
| interval | Type: Int, periodic execution | Default 24 |
| crontab | Type: String, execute Crontab expression regularly |    |
| recipients_part_one | Type: List[Object], recipient (first part), item attribute id/name |    |
| recipients_part_two | Type: List[Object], recipient (second part), item attribute id/name |    |
| obj_recipients_part_one | Type: List[Object], object storage recipient (first part), item attribute id/name |    |
| obj_recipients_part_two | Type: List[Object], object storage recipient (second part), item attribute id/name |    |
| types | Type: String[], asset type | linux, windows, unix, other, general, switch, "router", "firewall", "mysql", "mariadb", "postgresql", "oracle", "sqlserver", "clickhouse", "mongodb", "redis", "public", "private", "k8s", "website" |
| date_created | Type: String[date], creation time |    |
| date_updated | Type: String[date], update time |    |
| created_by | Type: String, creator |    |
| comment | Type: String, remarks |    |


- **Request Example**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/accounts/account-backup-plans/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "types": ["linux", "windows", "unix", "other", "general", "switch", "router", "firewall", "mysql", "mariadb",
        "postgresql", "oracle", "sqlserver", "clickhouse", "mongodb", "redis", "public", "private", "k8s", "website"],
        "is_periodic": true,
        "interval": 24,
        "name": "test",
        "recipients_part_one": [{
            "id": "d1a02c44-e40b-41ac-884a-c44f3c664209"
        }],
        "crontab": "0 0 * * *",
        "comment": "abcs"
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

def create_account_backup_plans():
    url = f"{API_URL}/api/v1/accounts/account-backup-plans/"
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
        "types": ["linux", "windows", "unix", "other", "general", "switch", "router", "firewall", "mysql", "mariadb",
        "postgresql", "oracle", "sqlserver", "clickhouse", "mongodb", "redis", "public", "private", "k8s", "website"],
        "is_periodic": True,
        "interval": 24,
        "name": "test",
        "crontab": "0 0 * * *",
        "comment": "abcs"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("Account backup plan created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_account_backup_plans()
```

- **Use Case:**

Scenario: MLPS compliance requires database account passwords to be archived offline regularly. Create an automatic backup policy for production MySQL and PostgreSQL database accounts at 2:00 a.m. every day, then separately send the split backup-file passwords to two administrators to reduce the risk of disclosure by one person.

```sh
curl -X POST 'https://localhost/api/v1/accounts/account-backup-plans/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "Production database account backup",
        "types": ["mysql", "postgresql"],
        "is_periodic": true,
        "crontab": "0 2 * * *",
        "recipients_part_one": [{
            "id": "d1a02c44-e40b-41ac-884a-c44f3c664209"
        }],
        "recipients_part_two": [{
            "id": "5f8c1e33-9a27-4b06-8d12-3e7a90b1c554"
        }],
        "is_password_divided_by_email": true,
        "comment": "Waiting for guarantee compliance：Daily offline backup of production library account"
    }'
```


## /api/v1/accounts/account-backup-plans/{id}/
### DELETE
- **Description:**
Delete account backup policy

- **Request headers:**  

| key              | value                                      | Remarks                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | b96810faac725563304dada8c323c4fa061863d4 is the administrator’s token information.       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 is the organization ID. This ID number is the default organization: Default. If left blank, it defaults to the Default organization. |
| Content-Type    | application/json                        | Output in json format                                                       |

- **Path parameter (Path):**  

| Parameter name | Description | Default value |
| --- | --- | --- |
| id* | Type: String, backup policy ID | - |

> Note: Parameters with * are required.
**Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/accounts/account-backup-plans/37e4c30a-71ea-4b58-91f3-a692715dcf99/' \
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
PLAN_ID     = "your plan id"

def delete_account_backup_plans():
    url = f"{API_URL}/api/v1/accounts/account-backup-plans/{PLAN_ID}/"
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
        print("Account backup plan deleted successfully")
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    delete_account_backup_plans()
```

- **Use Case:**

Scenario: The test environment assets have been migrated offline as a whole, and the "test environment account backup" policy originally configured for it is no longer needed. The operations personnel first find the policy ID by name through the GET interface and delete it to avoid continuing to generate invalid backup emails.

```sh
curl -X DELETE 'https://localhost/api/v1/accounts/account-backup-plans/8c6a5b12-4d3e-4f7a-9b08-2e15d6c73a40/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```
