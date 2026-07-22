## /api/v1/accounts/account-secrets/{id}/

### GET

- **Description:**
Query the password/key of the specified account (account ciphertext)

> Note: Viewing passwords requires MFA authentication by default. If you want to call this interface directly through the API, you need to add the parameter `SECURITY_VIEW_AUTH_NEED_MFA=False` in the `/opt/jumpserver/config/config.txt` configuration file, and execute `jmsctl restart` to take effect after restarting the service.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Path parameters:**

| Parameter name | Type | Description | Is it required? |
| --- | --- | --- | --- |
| id | String | Account ID, which can be obtained through the `/api/v1/accounts/accounts/` interface | Yes |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| id | Type: string, account ID | UUID |
| name | Type: string, account name |    |
| username | Type: string, account username |    |
| secret_type | Type: Object, ciphertext type | {"value":"password","label":"password"} etc. |
| secret | Type: string, account password/key plain text | The core return fields of this interface |
| asset | Type: object, belonging asset | Contains asset ID, name, address, platform and other information |
| privileged | Type: Boolean, whether it is a privileged account |    |
| connectivity | Type: object, connectability | {"value":"ok","label":"success"} etc. |
| has_secret | Type: boolean, whether the ciphertext has been managed |    |
| is_active | Type: boolean, whether to enable |    |
| version | Type: int, ciphertext version | The version is incremented each time the encryption is changed. |
| source | Type: object, account source | local (database)/collected (collection), etc. |
| date_created | Type: String(date-time), creation time |    |
| date_updated | Type: string(date-time), update time |    |

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/accounts/account-secrets/1e28b088-c86c-41b7-a85b-29e15c7cc8cb/' \
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
ACCOUNT_ID  = "1e28b088-c86c-41b7-a85b-29e15c7cc8cb"

def get_account_secret(account_id):
    url = f"{API_URL}/api/v1/accounts/account-secrets/{account_id}/"
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
    result = get_account_secret(ACCOUNT_ID)
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: The database inspection script connects to db-mysql-01 every night before performing backup. It retrieves the password of the root account on the asset from JumpServer in real time according to the account ID to avoid hard-coding the password in the script (the account ID has been filtered by asset name and user name through `/api/v1/accounts/accounts/` in advance).

```sh
curl -s -X GET 'https://localhost/api/v1/accounts/account-secrets/f3a9c2d1-7b64-4e0a-9c3f-5d8e2a1b6c40/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>' | jq -r '.secret'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)
