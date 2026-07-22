# Practical Case: Retrieve Asset Account Passwords Without Hard-Coding

## Scene description

Operations scripts, scheduled tasks, and CI/CD pipelines often hard-code passwords in scripts, configuration files, or pipeline variables when connecting to servers and databases. After a password is rotated, it must be updated everywhere and may spread or leak with the source repository. This case shows how an external script can retrieve a password from the JumpServer API at runtime. The script stores only JumpServer authentication credentials locally (injected through environment variables), never stores business passwords, and works with password change plans to rotate asset account passwords regularly. Every run retrieves the latest password, so the password exists only in JumpServer.

## Preconditions

- JumpServer V4 is deployed and the machine running the script has access to the JumpServer's HTTPS port.
- The calling credentials have been obtained: API Key (AK/SK, avatar in the upper right corner of the web page - created in API Key) or Bearer Token. The caller needs to have viewing permissions for the target asset account.
- The target asset and its account have secrets hosted in the JumpServer (available with the `has_secret=true` filter of the account list interface).
- Method 1 (direct secret retrieval) requires MFA authentication by default. To call the API directly, add `SECURITY_VIEW_AUTH_NEED_MFA=False` to `/opt/jumpserver/config/config.txt`, then run `jmsctl restart`. For details, see the [Account Secret Query API](../accounts/accountsecrets.md).
- Method 2 (PAM integrated application) requires permission to create integrated applications. For interface details, see [Application Integration Interface Document](../pam/integration-application.md).
- Python complete example dependency: `pip install requests httpsig`.

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/accounts/accounts/` | Query the account list by asset and get the account ID |
| GET | `/api/v1/accounts/account-secrets/{id}/` | Get account secret text in real time by account ID (method 1) |
| POST | `/api/v1/accounts/integration-applications/` | Create a PAM integrated application (method 2) |
| GET | `/api/v1/accounts/integration-applications/{id}/secret/` | Obtain the application key of the integrated application (method 2) |
| GET | `/api/v1/accounts/integration-applications/account-secret/` | Obtain secret text by asset + account through integrated application (Method 2) |

> Field-level details for these endpoints are available in your environment's online API documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

### Method 1: Retrieve the Secret Directly (Simple, Suitable for Controlled Intranets)

#### Step 1: Query account list by assets

Call the account list interface and use `asset_id` to limit assets. You can add query parameters such as `username` and `has_secret` to further filter and get the account ID from the returned results.

```sh
curl -X GET 'https://localhost/api/v1/accounts/accounts/?asset_id=9266b1f8-f74d-482c-805a-6eed0e099a42&username=root&has_secret=true' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Each account object returned contains `id`, `name`, `username`, `secret_type`, `asset`, `privileged`, `is_active`, `version` and other fields. Note: This interface will not return `secret` plaintext (`secret` is a write-only field), you must use the returned `id` to go to the next step to obtain the secret.

#### Step 2: Get the secret text by account ID

Use the account ID from the previous step to call the account secret endpoint. The returned `secret` is the plaintext password or key, and `version` is the secret version (incremented after each password change).

```sh
curl -X GET 'https://localhost/api/v1/accounts/account-secrets/1e28b088-c86c-41b7-a85b-29e15c7cc8cb/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Return example (excerpt):

```json
{
    "id": "1e28b088-c86c-41b7-a85b-29e15c7cc8cb",
    "name": "root",
    "username": "root",
    "secret_type": {"value": "password", "label": "Password"},
    "secret": "Xy3#example-secret",
    "version": 3
}
```

> If it returns 403 or prompts that MFA is required, please check whether the `SECURITY_VIEW_AUTH_NEED_MFA` configuration in the precondition has taken effect.

### Method 2: Retrieve the Secret Through a PAM Integration Application (Recommended for Production)

Integrating an application is equivalent to issuing an independent "application identity" to each external system: only authorize the account range it requires, and use `ip_group` to restrict the source IP. Compared with directly using the administrator Token/AK, once the credentials are leaked, the impact is much smaller, and it is suitable for long-term scripts and pipelines in production environments.

#### Step 3: Create the integration app

Specify the application name, authorized account range, and source IP range allowed for access.

```sh
curl -X POST 'https://localhost/api/v1/accounts/integration-applications/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "cicd-runner",
        "accounts": {"ids": ["1e28b088-c86c-41b7-a85b-29e15c7cc8cb"]},
        "ip_group": ["192.168.1.0/24"],
        "is_active": true,
        "comment": "Dedicated to CI pipeline secret retrieval"
    }'
```

`name` and `accounts` in the request body are required; `ip_group` defaults to `["*"]` (regardless of source). In the production environment, it is recommended to converge to the export IP of the script machine. Please refer to the online API documentation for the internal value structure of `accounts`. Note the application `id` in the return.

#### Step 4: Get the app key for the integrated app

Use the application ID to call the one-time key interface to obtain the calling key of the application. Sensitive content will only be returned when called for the first time. Please save it properly immediately (such as writing to the Secret variable of CI). If it is lost, it needs to be regenerated.

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/9b6b5a8e-6a1f-4c2a-9d0e-3f5e8a7b1c2d/secret/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

#### Step 5: Obtain password by asset + account through integrated application

The external script uses the integration application's identity to call the unified secret endpoint and locate the target secret by asset and account.

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/account-secret/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

> The asset and account query parameters and the returned secret fields for this endpoint are not included in this repository's Swagger definition (the `IntegrationAccountSecret` response schema declares only `asset`, `asset_id`, `account`, and `account_id`). For actual calls, refer to `accounts_integration_applications_account_secret` in your environment's online documentation at `https://<JumpServer-address>/api/docs`, or view the built-in SDK example on the integration application details page.

### Use a Password Change Plan for Regular Rotation

JumpServer password change plans perform password rotation. Create a periodic plan under **PAM - Account Password Change** in the Web console (or through the API; see the [Password Change Plan API](../pam/changepwd.md)) to assign a new random password to the asset account regularly. After rotation, the account secret `version` is incremented automatically. Because the script retrieves the secret at runtime, it gets the latest password without modification. Password changes no longer require synchronized updates to scripts or configuration files.

## Complete sample code

The following Python script implements Method 1 end to end: locate the account by asset and username, retrieve the secret at runtime, and use the password to connect to the target. JumpServer AK/SK credentials are injected through environment variables, and no plaintext password appears in the code or repository.

```python
# -*- coding: utf-8 -*-
"""
External scripts can retrieve asset account passwords without hard-coding

process：Query asset account list -> Target account -> Get account password in real time
Authentication information is all injected through environment variables，No plain text passwords are stored in scripts and code repositories.。

Depends on installation：
    pip install requests httpsig

environment variables：
    JMS_URL         JumpServer Access address，Such as https://localhost
    JMS_KEY_ID      API Key of AccessKey ID
    JMS_KEY_SECRET  API Key of AccessKey Secret
    JMS_ORG_ID      organization ID，The default organization is 00000000-0000-0000-0000-000000000002
"""

import os
import sys
from datetime import datetime

import requests
from httpsig.requests_auth import HTTPSignatureAuth

API_URL    = os.environ.get("JMS_URL", "https://localhost")
KEY_ID     = os.environ.get("JMS_KEY_ID")
KEY_SECRET = os.environ.get("JMS_KEY_SECRET")
ORG_ID     = os.environ.get("JMS_ORG_ID", "00000000-0000-0000-0000-000000000002")

ASSET_ID   = "9266b1f8-f74d-482c-805a-6eed0e099a42"   # target asset ID
USERNAME   = "root"                                   # Target account username


def build_auth():
    """structure AK/SK Signature authentication object"""
    signature_headers = ['(request-target)', 'accept', 'date']
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = signature_headers
    )


def build_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Accept": "application/json",
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }


def get_account_id(asset_id, username):
    """step one：by assets + Username query account，Return to account ID"""
    url = f"{API_URL}/api/v1/accounts/accounts/"
    params = {
        "asset_id": asset_id,
        "username": username,
        "has_secret": True
    }
    response = requests.get(
        url, auth = build_auth(), headers = build_headers(),
        params = params
    )
    response.raise_for_status()
    data = response.json()
    # Returned with paging parameters {count, results}，Otherwise, the list may be returned directly，Both are compatible
    results = data.get("results") if isinstance(data, dict) else data
    if not results:
        raise LookupError(f"assets {asset_id} No account with managed password found {username}")
    return results[0]["id"]


def get_account_secret(account_id):
    """Step 2：Get account password in real time（Always the latest version）"""
    url = f"{API_URL}/api/v1/accounts/account-secrets/{account_id}/"
    response = requests.get(
        url, auth = build_auth(), headers = build_headers()
    )
    response.raise_for_status()
    return response.json()


def main():
    if not KEY_ID or not KEY_SECRET:
        print("Please pass environment variables first JMS_KEY_ID / JMS_KEY_SECRET Inject authentication information")
        sys.exit(1)

    try:
        account_id = get_account_id(ASSET_ID, USERNAME)
        secret_info = get_account_secret(account_id)
    except requests.HTTPError as e:
        status = e.response.status_code if e.response is not None else "?"
        if str(status) == "403":
            print("Password access denied（403）：Please confirm caller permissions，and config.txt Have you hit"
                  "Configuration SECURITY_VIEW_AUTH_NEED_MFA=False and restart the service")
        else:
            print(f"API Request failed（HTTP {status}）：{e}")
        sys.exit(2)
    except requests.RequestException as e:
        print(f"Network request exception：{e}")
        sys.exit(2)
    except LookupError as e:
        print(f"Data error：{e}")
        sys.exit(3)

    username = secret_info.get("username")
    secret   = secret_info.get("secret")
    version  = secret_info.get("version")
    if not secret:
        print("The endpoint returned successfully, but the secret is empty. Confirm that the account password is managed by JumpServer.")
        sys.exit(3)

    # Demo：Print only desensitized information。Never output plaintext passwords in production logs！
    print(f"Account: {username}  Secret version: {version}  Password: {secret[:2]}****")

    # Use the password obtained here to connect to the target asset，For example via paramiko connect SSH：
    # import paramiko
    # ssh = paramiko.SSHClient()
    # ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # ssh.connect('<Asset address>', 22, username, secret)
    # The password only exists in the memory of this process，Disappears when the process ends


if __name__ == "__main__":
    main()
```

## FAQ

**Q1: Calling `/api/v1/accounts/account-secrets/{id}/` returns 403 or prompts that MFA authentication is required? **

A: Viewing account secrets requires MFA authentication by default, but API calls cannot enter MFA interactively. Add `SECURITY_VIEW_AUTH_NEED_MFA=False` to `/opt/jumpserver/config/config.txt`, run `jmsctl restart`, and try again. This setting lowers the protection for secret retrieval, so enable it only on a controlled network and protect the calling credentials. If the endpoint still returns 403, verify that the caller can view the account and that `X-JMS-ORG` identifies the account's organization.

**Q2: How should I choose between direct secret retrieval and a PAM integration application?**

A: Direct secret retrieval (Method 1) is the simplest option, but it uses a user-level Token or AK. Leaked credentials expose all API permissions granted to that user, so this method suits controlled intranets, temporary troubleshooting, or small scripts. A PAM integration application (Method 2) gives each external system an independent identity, authorizes only the required accounts, and can restrict source IPs with `ip_group`. It can be disabled independently (`is_active=false`) without affecting other systems. Method 2 is recommended for long-running production scripts and CI pipelines.

**Q3: Why is there no password field in the return of `GET /api/v1/accounts/accounts/`? **

A: The `secret` field in an account object is write-only and can be submitted only when creating or updating an account. List and detail endpoints never return plaintext. To retrieve a secret, explicitly call `account-secrets` or the integration application's secret endpoint so access can be tightly controlled and audited. The process must therefore query the account ID first and then retrieve the secret; the two steps cannot be combined.

**Q4: After a password change plan rotates the password, can the running script receive the old password?**

A: No. The secret endpoint always returns the latest secret version (`version` increments after each password change). As long as the script retrieves the secret at runtime and does not cache passwords, rotation is transparent. Existing long-lived connections, such as database sessions in a connection pool, are not affected by a password change. After a disconnect, retrieve the secret again before reconnecting. Do not cache passwords in files or environment variables for reuse.
