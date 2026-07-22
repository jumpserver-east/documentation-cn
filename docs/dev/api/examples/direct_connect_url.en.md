# Practical case: Generating URLs for users to directly connect to assets without passwords

## Scene description

The internal operations platform or CMDB platform of the enterprise usually hopes to embed JumpServer as the underlying session gateway into its own page: when the user clicks on an asset on the external platform, the Web terminal session of JumpServer will be automatically opened as the user, without the need to manually log in to JumpServer. In this case, through the API call as an administrator, an asset access token is first created for the target user, and then a one-time password-free login URL is generated for the user. The external platform only needs the `window.open` URL to complete the two-step action of "password-free login + direct asset connection".

## Preconditions

1. **Enable SSO authentication function**: Modify the `/opt/jumpserver/config/config.txt` configuration file, add the following two lines, and execute `jmsctl restart` to restart the service to take effect:

    ```text
    AUTH_SSO=True
    AUTH_SSO_AUTHKEY_TTL=900
    ```

    Among them, `AUTH_SSO_AUTHKEY_TTL` is the validity period (seconds) of the authkey for password-free login.

2. **API Credentials for Administrators**: The following interfaces require administrator privileges. Please prepare the administrator's Bearer Token, or the API Key (AccessKey ID/Secret, used with HTTP Signature signature authentication) created on the personal information page.
3. **The target user has been authorized for the target asset**: The user must have obtained the connection permission for the asset and the corresponding account through the asset authorization rules, otherwise the asset access Token cannot be created.
4. The username of the target user (such as `zhangsan`) and the name of the target asset (such as `web-server-01`) are known and used to query their respective IDs.

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/users/users/?username=<username>` | Query users by username and get the user ID |
| GET | `/api/v1/assets/assets/?name=<asset-name>` | Query assets by name and get the asset ID |
| POST | `/api/v1/authentication/super-connection-token/` | Create asset access token (super connection token) for the user |
| POST | `/api/v1/authentication/sso/login-url/` | Generate a one-time password-free login URL for the user |

> Field-level details for each endpoint are available in your JumpServer online API documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

The request headers in the following examples are unified as:

| key | value | Remarks |
| --- | --- | --- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Administrator’s authentication token, the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID, the example is the default organization Default, if left blank, the default is the Default organization |
| Content-Type | `application/json` | The request/response body is in JSON format |

### Step 1: Query user ID

Filter the user list accurately by user name, take the `id` field (UUID) from the returned result, and use it as the `user` parameter when creating a Token later.

```sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 2: Query asset ID

Filter the asset list by asset name, take the `id` field (UUID) from the returned result, and use it as the `asset` parameter later.

```sh
curl -X GET 'https://localhost/api/v1/assets/assets/?name=web-server-01' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 3: Create asset access token for the user

Call `POST /api/v1/authentication/super-connection-token/` to create an asset access token for the target user as an administrator.

Request body parameter description:

| Parameter name | Type | Description | Is it required? | Remarks |
| --- | --- | --- | --- | --- |
| user | String | User ID | Yes | The user UUID queried in the first step |
| asset | String | Asset ID | Yes | The asset UUID queried in the second step |
| account | String | Account | Yes | For a managed account, enter its username (such as `root`); to enter an account manually, use `@INPUT` |
| protocol | String | connection protocol | No | If not passed, the default is `ssh`; such as `rdp`, the protocol needs to be enabled for the asset |
| connect_method | String | Connection method | Yes | `web_cli` (character web terminal), `web_gui` (graphical web terminal), `ssh_client`, `mstsc` |
| connect_options | Object | Connection parameters | No | Such as `{"resolution": "1920x1080"}` to set the graphics session resolution |
| input_username | String | Manually entered account username | No | Only used when `account` is `@INPUT` |
| input_secret | String | Manually entered account password | No | Only used when `account` is `@INPUT` |

```sh
curl -X POST 'https://localhost/api/v1/authentication/super-connection-token/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "user": "53966f1e-ffbc-4f4c-8a10-a6bc70000a98",
            "asset": "fb0c515b-ce80-4391-80b2-5440e5de8133",
            "account": "root",
            "protocol": "ssh",
            "connect_method": "web_cli",
            "connect_options": {
                "resolution": "1920x1080"
            }
        }'
```

Return example (excerpt):

```json
{
    "id": "17a52559-fb2e-4262-8887-fd58020aaa5c",
    "value": "IgJ6aO6wo7GTyAIB",
    "user": {"id": "53966f1e-ffbc-4f4c-8a10-a6bc70000a98", "name": "zhangsan"},
    "asset": {"id": "fb0c515b-ce80-4391-80b2-5440e5de8133", "name": "web-server-01"},
    "account": "root",
    "protocol": "ssh",
    "connect_method": "web_cli",
    "expire_time": 299,
    "is_expired": false,
    "date_expired": "2026/07/22 20:27:53 +0800"
}
```

Among them, `id` is the `token_id` to be used in the next step; `expire_time` is the remaining validity period of the Token (in seconds). Please complete the subsequent steps within the validity period.

### Step 4: Generate a password-free login URL for the user

Call `POST /api/v1/authentication/sso/login-url/` to generate a one-time password-free login URL for the target user.

Request body parameter description:

| Parameter name | Type | Description | Is it required? | Remarks |
| --- | --- | --- | --- | --- |
| username | String | JumpServer username | Yes | Such as `zhangsan`. **Note that the field name is `username`, not `user`** (Some old information is written as `user`, which is wrong) |
| next | String | Jump address after successful login | No | If not passed, you will enter the JumpServer default homepage after logging in; in this case, it must be passed in, and the values are shown in the table below. |

Commonly used values for the `next` parameter:

| value | Description |
| --- | --- |
| `/koko/connect/?token={token_id}` | Open character Web terminal (SSH/Telnet, etc., KoKo component) inline, `token_id` is the `id` returned in the third step |
| `/lion/connect/?token={token_id}` | Open the graphical web terminal inline (RDP/VNC, etc., Lion component), `token_id` is the `id` returned in the third step |
| `/luna/?login_to={asset_id}&type=asset` | Open the Luna complete web terminal page and locate the specified asset. `asset_id` is the asset UUID |

```sh
curl -X POST 'https://localhost/api/v1/authentication/sso/login-url/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "username": "zhangsan",
            "next": "/koko/connect/?token=17a52559-fb2e-4262-8887-fd58020aaa5c"
        }'
```

Return example:

```json
{
    "login_url": "https://localhost/api/v1/authentication/sso/login/?authkey=27fe6e65-e1f3-4b9d-9a9a-5aeecad8cf78&next=%2Fkoko%2Fconnect%2F%3Ftoken%3D17a52559-fb2e-4262-8887-fd58020aaa5c"
}
```

### Step 5: Open the URL on the external platform

The front end of the external platform can directly `window.open(login_url)` (or jump to a new tab page): when the browser accesses the URL, it will automatically complete the password-free login of the target user, and then jump to the Web terminal page specified by `next` to directly connect to the target asset. The `authkey` in `login_url` is a one-time credential and is subject to the validity period of `AUTH_SSO_AUTHKEY_TTL`. It should be used immediately and should not be cached.

## Complete sample code

The following Python script strings together the complete process: check user ID → check asset ID → create asset access token → generate password-free login URL. The authentication method uses API Key (AK/SK) + HTTP Signature signature (depends on two packages `requests` and `httpsig`: `pip install requests httpsig`).

```python
# Generate users’ password-free direct-connect assets URL —— Complete example
# process：Check user ID -> Check assets ID -> Create asset access Token -> Generate password-free login URL

import json
import sys
from datetime import datetime

import requests
from httpsig.requests_auth import HTTPSignatureAuth

API_URL    = "https://localhost"
KEY_ID     = "your id"          # Administrator API Key of AccessKey ID
KEY_SECRET = "your secret"      # Administrator API Key of AccessKey Secret
ORG_ID     = "00000000-0000-0000-0000-000000000002"  # organization ID，Default organization Default

USERNAME       = "zhangsan"        # Need to directly connect assets without confidentiality JumpServer Username
ASSET_NAME     = "web-server-01"   # Target asset name
ACCOUNT        = "root"            # Managed account username on the asset; use "@INPUT" for manual entry
PROTOCOL       = "ssh"             # connection protocol
CONNECT_METHOD = "web_cli"         # For character protocol web_cli（koko）；For graphics protocol web_gui（lion）


def api_request(method, path, params=None, payload=None):
    """Generic request encapsulation with signature authentication，Exception thrown when request fails"""
    url = f"{API_URL}{path}"
    headers = {
        "Accept": "application/json",
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime("%a, %d %b %Y %H:%M:%S GMT"),
    }
    auth = HTTPSignatureAuth(
        key_id=KEY_ID, secret=KEY_SECRET,
        algorithm="hmac-sha256",
        headers=['(request-target)', 'accept', 'date'],
    )
    response = requests.request(
        method, url, auth=auth, headers=headers,
        params=params,
        data=json.dumps(payload) if payload is not None else None,
    )
    response.raise_for_status()
    return response.json()


def get_user_id(username):
    """first step：Query users by username ID"""
    data = api_request("GET", "/api/v1/users/users/", params={"username": username})
    users = data["results"] if isinstance(data, dict) else data
    if not users:
        raise ValueError(f"User not found: {username}")
    return users[0]["id"]


def get_asset_id(asset_name):
    """Step 2：Query assets by name ID"""
    data = api_request("GET", "/api/v1/assets/assets/", params={"name": asset_name})
    assets = data["results"] if isinstance(data, dict) else data
    if not assets:
        raise ValueError(f"Asset not found: {asset_name}")
    return assets[0]["id"]


def create_connection_token(user_id, asset_id):
    """Step 3：Create asset access for users Token，Return token_id"""
    payload = {
        "user": user_id,
        "asset": asset_id,
        "account": ACCOUNT,
        "protocol": PROTOCOL,
        "connect_method": CONNECT_METHOD,
        "connect_options": {"resolution": "1920x1080"},
        # Manually enter account scenario（ACCOUNT = "@INPUT"）When adding the following two fields：
        # "input_username": "root",
        # "input_secret": "<Account password>",
    }
    token = api_request(
        "POST", "/api/v1/authentication/super-connection-token/", payload=payload
    )
    print(f"Token Created successfully, id={token['id']}, Remaining validity period {token['expire_time']} seconds")
    return token["id"]


def create_login_url(username, next_url):
    """Step 4：Generate one-time password-free login for users URL。Note that the field name is username，No user"""
    payload = {"username": username, "next": next_url}
    data = api_request(
        "POST", "/api/v1/authentication/sso/login-url/", payload=payload
    )
    return data["login_url"]


def main():
    try:
        user_id = get_user_id(USERNAME)
        print(f"User {USERNAME} of ID: {user_id}")

        asset_id = get_asset_id(ASSET_NAME)
        print(f"assets {ASSET_NAME} of ID: {asset_id}")

        token_id = create_connection_token(user_id, asset_id)

        # character protocol go koko component page；graphics protocol（web_gui）Please change to /lion/connect/
        next_url = f"/koko/connect/?token={token_id}"
        login_url = create_login_url(USERNAME, next_url)

        print("Direct connection without density URL（external platform window.open to open）:")
        print(login_url)
    except requests.HTTPError as e:
        print(f"API Request failed: {e}, Response content: {e.response.text}")
        sys.exit(1)
    except (requests.RequestException, ValueError, KeyError) as e:
        print(f"Execution error: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

## FAQ

**Q1: Calling `/api/v1/authentication/sso/login-url/` reports 400, prompting that the `username` field is required? **

A: The request body fields of this interface are `username` and `next`. In some old materials, it is written as `user`, which is wrong. Please check whether the request body field name is `username`.

**Q2: After opening `login_url`, there is no password-free login, or it prompts authentication failure? **

A: There are two common reasons: one is that `AUTH_SSO=True` is not configured in `/opt/jumpserver/config/config.txt` and the service is restarted; the other is that the `authkey` in `login_url` is a one-time credential and is limited by `AUTH_SSO_AUTHKEY_TTL` (seconds). It will become invalid when it times out or has been used. The interface needs to be re-called to generate, do not cache and reuse it.

**Q3: When creating an asset access token, an error occurs, indicating that the account does not exist or does not have permission? **

A: The value of the `account` field depends on the scenario - for the managed account, fill in the account username (such as `root`) on the asset, and for manually entering the account, fill in `@INPUT` and pass in `input_username` and `input_secret` at the same time. In addition, the target user must have obtained the connection permission for the asset and the corresponding account through asset authorization rules. The administrator cannot bypass authorization when creating a Token for the user.

**Q4: When the generated URL is opened, it is prompted that the token has expired? **

A: The asset access token has a short validity period (the return value `expire_time` is the remaining seconds, the default is about 5 minutes) and is used once by default (`is_reusable` is false). It is recommended that the interface be called in real time to generate Token and `login_url` when the user clicks on the asset, so that they can be used immediately and do not generate them in batches in advance.
