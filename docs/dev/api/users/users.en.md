## /api/v1/users/users/

### GET

- **Description:**
Get user list

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, user data list | List elements are user objects (see below) |
| id | Type: string, user ID | UUID |
| name | Type: string, display name |    |
| username | Type: string, username | Login name |
| email | Type: string, email |    |
| mfa_level | Type: object, MFA level | {"value":0,"label":"disabled"} etc. |
| source | Type: object, user source | local/ldap/openid/radius/cas/saml2/oauth2/custom |
| wecom_id | Type: string, corporate WeChat ID | Exists when binding |
| dingtalk_id | Type: string, DingTalk ID | Exists when binding |
| feishu_id | Type: string, Feishu ID | Exists when binding |
| created_by | Type: string, creator |    |
| updated_by | Type: string, updater |    |
| comment | Type: string, remarks |    |
| groups | Type: list, user group | List of objects or IDs |
| system_roles | Type: list, system role | Contains "user" role by default |
| org_roles | Type: list, organization role | Default includes "Organization User" |
| password_strategy | Type: object, password policy | Shaped like {"value":"email","label":"..."}; can be null, default value=email (reset link sent by email) |
| is_service_account | Type: boolean, whether to component account | true indicates the system internal account |
| is_valid | Type: boolean, valid or not |    |
| is_expired | Type: boolean, whether to expire |    |
| is_active | Type: boolean, whether to enable |    |
| is_otp_secret_key_bound | Type: boolean, whether to bind OTP key |    |
| can_public_key_auth | Type: boolean, whether to allow SSH public key |    |
| mfa_enabled | Type: boolean, whether to enable MFA |    |
| need_update_password | Type: boolean, password needs to be changed next time you log in |    |
| mfa_force_enabled | Type: boolean, whether to force MFA |    |
| is_first_login | Type: boolean, whether to log in for the first time |    |
| login_blocked | Type: boolean, login blocking |    |
| date_expired | Type: string(date-time), expiration time |    |
| date_joined | Type: string(date-time), add time |    |
| last_login | Type: string(date-time), last login time |    |
| date_updated | Type: string(date-time), update time |    |
| date_password_last_updated | Type: string(date-time), password update time |    |

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/users/users/?offset=0&limit=15' \
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

def get_user_info():
    url = f"{API_URL}/api/v1/users/users/"
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
    result = get_user_info()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: Before the quarterly security audit, operations needs to pull a list of all deactivated accounts that come from LDAP and check whether domain account recycling is missing.

```sh
curl -X GET 'https://localhost/api/v1/users/users/?source=ldap&is_active=false&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)

### POST

- **Description:**
Create user

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID, leave blank to default to `Default` organization |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Request body parameters (Body):**

| Parameter name | Description | Optional values / remarks |
| --- | --- | --- |
| name* | Type: string, name/display name | - |
| username* | Type: string, username (login) | - |
| email* | Type: string, email | - |
| wechat | Type: string, WeChat | - |
| phone | Type: string, mobile phone | - |
| groups | Type: string[], list of user group IDs | - |
| password | Type: string, password (required when password_strategy=custom) | - |
| need_update_password | Type: boolean, whether the password needs to be changed next time you log in | Default false;[true,false] |
| public_key | Type: string, SSH public key | - |
| system_roles | Type: object[]/string[], system role | Element contains pk ID |
| org_roles | Type: object[]/string[], organization role | Element contains pk ID |
| password_strategy | Type: string, password policy | email/custom; email=mail setting password; string value (such as "email"/"custom") is passed in the request and object is returned in the response |
| source | Type: string, user source | Default local; optional local/ldap/ldap_ha/openid/radius/cas/saml2/oauth2/wecom/dingtalk/feishu/lark/slack/custom |
| mfa_level | Type: integer, MFA level | 0=disabled 1=enabled 2=mandatory |
| date_expired | Type: string(date-time), user expiration time | For example: 2023-02-04T00:54:39.000Z |

> Note: Parameters with * are required.
- **Return Parameters:** (Successful creation returns a complete user object, consistent with the structure of a single element in the GET list)

| Field name | Description | Remarks |
| --- | --- | --- |
| id | Type: string, user ID | UUID |
| name | Type: string, bastion host user name |    |
| username | Type: string, bastion host user name | Login name |
| email | Type: string, email |    |
| wechat | Type: string, WeChat |    |
| phone | Type: string, phone number |    |
| mfa_level | Type: object, MFA level | {"value":0,"label":"disabled"} etc. |
| source | Type: object, user source | local/ldap/.../custom |
| wecom_id | Type: string, corporate WeChat ID | Exists when binding |
| dingtalk_id | Type: string, DingTalk ID | Exists when binding |
| feishu_id | Type: string, Feishu ID | Exists when binding |
| created_by | Type: string, creator |    |
| updated_by | Type: string, updater |    |
| comment | Type: string, remarks |    |
| groups | Type: list, user group | List of objects or IDs |
| system_roles | Type: list, system role | Contains "user" role by default |
| org_roles | Type: list, organization role | Default includes "Organization User" |
| password_strategy | Type: object, password policy | Shaped like {"value":"email","label":"..."}; can be null, default value=email (reset link sent by email) |
| is_service_account | Type: boolean, whether to component account | true indicates the system internal account |
| is_valid | Type: boolean, valid or not |    |
| is_expired | Type: boolean, whether to expire |    |
| is_active | Type: boolean, whether to enable |    |
| is_otp_secret_key_bound | Type: boolean, whether to bind OTP key |    |
| can_public_key_auth | Type: boolean, whether to allow SSH public key |    |
| mfa_enabled | Type: boolean, whether to enable MFA |    |
| need_update_password | Type: boolean, password needs to be changed next time you log in |    |
| mfa_force_enabled | Type: boolean, whether to force MFA |    |
| is_first_login | Type: boolean, whether to log in for the first time |    |
| login_blocked | Type: boolean, login blocking |    |
| date_expired | Type: string(date-time), expiration time |    |
| date_joined | Type: string(date-time), add time |    |
| last_login | Type: string(date-time), last login time |    |
| date_updated | Type: string(date-time), update time |    |
| date_password_last_updated | Type: string(date-time), password update time |    |

- **Request Example**

**CURL**

``` sh
curl -X POST 'https://localhost/api/v1/users/users/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ 
            "name": "api_test",
            "username": "api_test",
            "password": "apitest",
            "password_strategy":"custom", 
            "email":"api_test@fit2cloud.com", 
            "mfa_level":0, 
            "source":"local", 
            "system_roles":[ 
                {"pk":"00000000-0000-0000-0000-000000000003"} 
            ], 
            "org_roles":[ 
                {"pk":"00000000-0000-0000-0000-000000000007"} 
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

def create_user():
    url = f"{API_URL}/api/v1/users/users/"
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
        "name": "api_test",
        "username": "api_test",
        "password": "apitest",
        "password_strategy": "custom",
        "email": "api_test@fit2cloud.com",
        "mfa_level": 0,
        "source": "local",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("User created successfully:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    create_user()
```

- **Use Case:**

Scenario: New employee Zhang San joins the operations team, and the personnel system triggers the account opening process: Create a local account, send a password setting link via email, enable MFA, and set the account expiration time according to the contract period.

```sh
curl -X POST 'https://localhost/api/v1/users/users/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "Zhang San",
        "username": "zhangsan",
        "email": "zhangsan@example.com",
        "phone": "13800138000",
        "password_strategy": "email",
        "source": "local",
        "mfa_level": 1,
        "date_expired": "2027-07-21T16:00:00.000Z"
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)

## /api/v1/users/users/{id}/

### GET

- **Description:**
Get user details

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | Authentication Token, administrator or user with viewing permissions |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID, default is Default |
| Content-Type | `application/json` | Response body is JSON |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string(UUID), user ID | Yes |

- **Return parameters:** (The structure is consistent with creating/listing a single element)

| Field name | Description | Remarks |
| --- | --- | --- |
| id | Type: string, user ID | UUID |
| name | Type: string, bastion host user name |    |
| username | Type: string, bastion host user name | Login name |
| email | Type: string, email |    |
| wechat | Type: string, WeChat |    |
| phone | Type: string, phone number |    |
| mfa_level | Type: object, MFA level | {"value":0,"label":"disabled"} etc. |
| source | Type: object, user source | local/ldap/.../custom |
| wecom_id | Type: string, corporate WeChat ID | Exists when binding |
| dingtalk_id | Type: string, DingTalk ID | Exists when binding |
| feishu_id | Type: string, Feishu ID | Exists when binding |
| created_by | Type: string, creator |    |
| updated_by | Type: string, updater |    |
| comment | Type: string, remarks |    |
| groups | Type: list, user group | List of objects or IDs |
| system_roles | Type: list, system role | Contains "user" role by default |
| org_roles | Type: list, organization role | Default includes "Organization User" |
| password_strategy | Type: object, password policy | Shaped like {"value":"email","label":"..."}; can be null, default value=email (reset link sent by email) |
| is_service_account | Type: boolean, whether to component account |    |
| is_valid | Type: boolean, valid or not |    |
| is_expired | Type: boolean, whether to expire |    |
| is_active | Type: boolean, whether to enable |    |
| is_otp_secret_key_bound | Type: boolean, whether to bind OTP key |    |
| can_public_key_auth | Type: boolean, whether to allow SSH public key |    |
| mfa_enabled | Type: boolean, whether to enable MFA |    |
| need_update_password | Type: boolean, password needs to be changed next time you log in |    |
| mfa_force_enabled | Type: boolean, whether to force MFA |    |
| is_first_login | Type: boolean, whether to log in for the first time |    |
| login_blocked | Type: boolean, login blocking |    |
| date_expired | Type: string(date-time), expiration time |    |
| date_joined | Type: string(date-time), add time |    |
| last_login | Type: string(date-time), last login time |    |
| date_updated | Type: string(date-time), update time |    |
| date_password_last_updated | Type: string(date-time), password update time |    |

- **Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Authorization: Bearer <token>' \
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
USER_ID     = "your user id"

def get_user_info():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
    result = get_user_info()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: User zhangsan submits a work order and reports that he cannot log in to the bastion host. The administrator queries the details according to his user ID and checks the `is_active`, `is_expired`, `login_blocked` and other fields to locate the reason.

```sh
curl -X GET 'https://localhost/api/v1/users/users/3f7b9c2e-8d41-4a5b-9c6d-1e2f3a4b5c6d/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)

### PUT

- **Description:**
Update user

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | Authentication Token, which requires modification permissions |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID, default Default |
| Content-Type | `application/json` | Request/response body is JSON |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string(UUID), user ID | Yes |

- **Request body parameters (Body):**

| Parameter name | Description | Optional values / remarks |
| --- | --- | --- |
| name* | Type: string, name/display name | - |
| username* | Type: string, username (login) | - |
| email* | Type: string, email | - |
| wechat | Type: string, WeChat | - |
| phone | Type: string, mobile phone | - |
| groups | Type: string[], list of user group IDs | - |
| password | Type: string, password | Required when password_strategy=custom |
| need_update_password | Type: boolean, password needs to be changed next time you log in | Default false |
| public_key | Type: string, SSH public key | - |
| system_roles | Type: object[]/string[], system role | Element contains pk |
| org_roles | Type: object[]/string[], organization role | Element contains pk |
| password_strategy | Type: string, password policy | email/custom; pass a string value (such as "email"/"custom") in the request, and return object in the response |
| source | Type: string, user source | local/ldap/ldap_ha/openid/radius/cas/saml2/oauth2/wecom/dingtalk/feishu/lark/slack/custom |
| mfa_level | Type: integer, MFA level | 0=disabled 1=enabled 2=mandatory |
| date_expired | Type: string(date-time), user expiration time | 2023-02-04T00:54:39.000Z |

> Note: Parameters with * are required.
- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| id | Type: string, user ID | UUID |
| name | Type: string, bastion host user name |    |
| username | Type: string, bastion host user name | Login name |
| email | Type: string, email |    |
| wechat | Type: string, WeChat |    |
| phone | Type: string, phone number |    |
| mfa_level | Type: object, MFA level | {"value":0,"label":"disabled"} etc. |
| source | Type: object, user source | local/ldap/.../custom |
| wecom_id | Type: string, corporate WeChat ID | Exists when binding |
| dingtalk_id | Type: string, DingTalk ID | Exists when binding |
| feishu_id | Type: string, Feishu ID | Exists when binding |
| created_by | Type: string, creator |    |
| updated_by | Type: string, updater |    |
| comment | Type: string, remarks |    |
| groups | Type: list, user group | List of objects or IDs |
| system_roles | Type: list, system role | Contains "user" role by default |
| org_roles | Type: list, organization role | Default includes "Organization User" |
| password_strategy | Type: object, password policy | Shaped like {"value":"email","label":"..."}; can be null, default value=email (reset link sent by email) |
| is_service_account | Type: boolean, whether to component account |    |
| is_valid | Type: boolean, valid or not |    |
| is_expired | Type: boolean, whether to expire |    |
| is_active | Type: boolean, whether to enable |    |
| is_otp_secret_key_bound | Type: boolean, whether to bind OTP key |    |
| can_public_key_auth | Type: boolean, whether to allow SSH public key |    |
| mfa_enabled | Type: boolean, whether to enable MFA |    |
| need_update_password | Type: boolean, password needs to be changed next time you log in |    |
| mfa_force_enabled | Type: boolean, whether to force MFA |    |
| is_first_login | Type: boolean, whether to log in for the first time |    |
| login_blocked | Type: boolean, login blocking |    |
| date_expired | Type: string(date-time), expiration time |    |
| date_joined | Type: string(date-time), add time |    |
| last_login | Type: string(date-time), last login time |    |
| date_updated | Type: string(date-time), update time |    |
| date_password_last_updated | Type: string(date-time), password update time |    |

- **Request Example**

**CURL**
```sh
curl -X PUT 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "new_name",
        "username": "new_username",
        "password_strategy": "email",
        "email": "user@fit2cloud.com",
        "mfa_level": 1,
        "source": "local",
        "date_expired": "2093-02-05T08:28:41.726694Z",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
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
USER_ID     = "your user id"

def update_user():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
        "name": "api_test",
        "username": "api_test_update",
        "password": "apitest",
        "password_strategy": "custom",
        "email": "api_test@fit2cloud.com",
        "mfa_level": 0,
        "source": "local",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
    }

    try:
        response = requests.put(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("User update successful:")
        print(json.dumps(response.json(), indent=2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    update_user()
```

- **Use Case:**

Scenario: Employee Li Si is transferred from the test group to the operations group, and his display name, email address, and user group are fully updated, while the account validity period is extended and the default role is retained.

```sh
curl -X PUT 'https://localhost/api/v1/users/users/9a8b7c6d-5e4f-4a3b-8c1d-0e9f8a7b6c5d/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "name": "John Doe",
        "username": "lisi",
        "email": "lisi@example.com",
        "groups": ["c1d2e3f4-a5b6-4c7d-8e9f-0a1b2c3d4e5f"],
        "password_strategy": "email",
        "source": "local",
        "mfa_level": 1,
        "date_expired": "2028-12-31T16:00:00.000Z",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)

### PATCH

- **Description:**
Partially updated users

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | Authentication Token, which requires modification permissions |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID, default Default |
| Content-Type | `application/json` | Request/response body is JSON |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string(UUID), user ID | Yes |

- **Request body (Body optional field):**

| Parameter name | Description | Remarks |
| --- | --- | --- |
| name | Type: string, name | Optional |
| email | Type: string, email | Optional |
| wechat | Type: string, WeChat | Optional |
| phone | Type: string, mobile phone | Optional |
| groups | Type: string[], list of user group IDs | Replace the entire list |
| password | Type: string, new password | Used when password_strategy=custom |
| need_update_password | Type: boolean, password needs to be changed next time you log in |    |
| public_key | Type: string, SSH public key |    |
| system_roles | Type: object[]/string[], system role | Full replacement |
| org_roles | Type: object[]/string[], organization role | Full replacement |
| password_strategy | Type: string, password policy | email/custom; pass a string value (such as "email"/"custom") in the request, and return object in the response |
| source | Type: string, user source |    |
| mfa_level | Type: integer, MFA level | 0/1/2 |
| date_expired | Type: string(date-time), user expiration time |    |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| id | Type: string, user ID | UUID |
| name | Type: string, bastion host user name |    |
| username | Type: string, bastion host user name | Login name |
| email | Type: string, email |    |
| wechat | Type: string, WeChat |    |
| phone | Type: string, phone number |    |
| mfa_level | Type: object, MFA level | {"value":0,"label":"disabled"} etc. |
| source | Type: object, user source | local/ldap/.../custom |
| wecom_id | Type: string, corporate WeChat ID | Exists when binding |
| dingtalk_id | Type: string, DingTalk ID | Exists when binding |
| feishu_id | Type: string, Feishu ID | Exists when binding |
| created_by | Type: string, creator |    |
| updated_by | Type: string, updater |    |
| comment | Type: string, remarks |    |
| groups | Type: list, user group | List of objects or IDs |
| system_roles | Type: list, system role | Contains "user" role by default |
| org_roles | Type: list, organization role | Default includes "Organization User" |
| password_strategy | Type: object, password policy | Shaped like {"value":"email","label":"..."}; can be null, default value=email (reset link sent by email) |
| is_service_account | Type: boolean, whether to component account |    |
| is_valid | Type: boolean, valid or not |    |
| is_expired | Type: boolean, whether to expire |    |
| is_active | Type: boolean, whether to enable |    |
| is_otp_secret_key_bound | Type: boolean, whether to bind OTP key |    |
| can_public_key_auth | Type: boolean, whether to allow SSH public key |    |
| mfa_enabled | Type: boolean, whether to enable MFA |    |
| need_update_password | Type: boolean, password needs to be changed next time you log in |    |
| mfa_force_enabled | Type: boolean, whether to force MFA |    |
| is_first_login | Type: boolean, whether to log in for the first time |    |
| login_blocked | Type: boolean, login blocking |    |
| date_expired | Type: string(date-time), expiration time |    |
| date_joined | Type: string(date-time), add time |    |
| last_login | Type: string(date-time), last login time |    |
| date_updated | Type: string(date-time), update time |    |
| date_password_last_updated | Type: string(date-time), password update time |    |

- **Request Example**

**CURL**
```sh
curl -X PATCH 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ "name": "partial_update_name" }'
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
USER_ID     = "your user id"

def partial_update_user():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
        "username": "api_test_update"
    }

    try:
        response = requests.patch(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("User update successful:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"Error:{e}")

if __name__ == "__main__":
    partial_update_user()
```

- **Use Case:**

Scenario: Security rectification requires that the account zhaoliu with high-risk permissions be forced to enable MFA, and require him to change his password the next time he logs in, while the rest of the information remains unchanged.

```sh
curl -X PATCH 'https://localhost/api/v1/users/users/7e6d5c4b-3a2f-4e1d-9c8b-7a6f5e4d3c2b/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' \
    -d '{
        "mfa_level": 2,
        "need_update_password": true
    }'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)

### DELETE

- **Description:**
Delete user

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | Authentication Token, need to have delete permission |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID, default Default |

- **Path Params:**

| Name | Description | Required |
| --- | --- | --- |
| id | Type: string(UUID), user ID | Yes |

- **Return:** 204 No Content (no response body if deleted successfully)

- **Request Example**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Authorization: Bearer <token>' \
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
USER_ID     = "your user id"

def delete_user():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
        print(f"User deleted successfully: {response.status_code}")
    except Exception as e:
        print(f"API Request failed:{e}")
        return None

if __name__ == "__main__":
    delete_user()
```

- **Use Case:**

Scenario: Employee Wang Wu resigns. After the final review of the resignation work order is passed, an automated script will delete his bastion host account and complete the recovery of access rights.

```sh
curl -X DELETE 'https://localhost/api/v1/users/users/5b4a3c2d-1e0f-4d9c-8b7a-6f5e4d3c2b1a/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: User Lifecycle Automation](../examples/user_lifecycle.md)
