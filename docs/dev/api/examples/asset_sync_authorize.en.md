# Practical case: external system applies for assets and automatically authorizes them

## Scene description

The enterprise's internal CMDB or operations platform (hereinafter referred to as "external system") needs to regularly pull asset and node lists from JumpServer and display them to business users for browsing and retrieval. A user initiates a use application for an asset in an external system. After the external system completes its own approval process, it calls the JumpServer interface to create a **time-limited asset authorization** for the user (control the validity and expiration time through `date_start` / `date_expired`). The authorization will automatically expire after expiration, without the need for manual recycling, thereby realizing a complete closed loop of "application → approval → authorization → automatic expiration".

## Preconditions

- Have an API authentication credential with administrative rights: the administrator's Bearer Token, or the API Key (AK/SK, used with HTTP signature authentication) created in the system settings. This credential requires the asset, the user's view permissions, and the asset authorization create permissions.
- Clarify the organization where the asset and user are located, and specify the organization ID through the request header `X-JMS-ORG` when calling the interface (if not passed, the default is `Default` organization).
- The applicant already exists in JumpServer (local user or user synchronized through LDAP, etc.), and the external system can get its username (`username`) for reverse checking of the user ID.
- The assets being applied for have been managed on JumpServer, and available login accounts have been configured on the assets (otherwise, there is no actual connectable account for the authorized `accounts`).
- The complete Python example requires installation of dependencies: `pip install requests httpsig`.

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/assets/nodes/` | Obtain the node (asset tree) list for synchronized display by external systems |
| GET | `/api/v1/assets/assets/` | Get the asset list in pages (supports filtering such as `search`/`name`/`address`) for synchronous display by external systems |
| GET | `/api/v1/assets/nodes/{id}/assets/` | Get the asset list under the specified node (used when the external system is displayed by node tree) |
| GET | `/api/v1/users/users/` | Press `username` to check the applicant’s user ID in JumpServer |
| POST | `/api/v1/perms/asset-permissions/` | Create a limited-time asset authorization after approval (`date_start` / `date_expired`) |
| GET | `/api/v1/perms/users/{user}/assets/` | Query the user's currently authorized assets for back verification of authorization results |
| DELETE | `/api/v1/perms/asset-permissions/{id}/` | (Optional) Delete authorization when approval is revoked or recycled in advance |

> Field-level details for these endpoints are available in the JumpServer online API documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

### Step 1: Synchronize node and asset list

The external system pulls the node and asset list regularly (such as every hour) and caches them in the local database for users to browse and initiate applications. First pull the node list and build the asset tree structure.

```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/?offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Then pull the asset list in pages. The `search` parameter can be fuzzy filtered by name and address:

```sh
curl -X GET 'https://localhost/api/v1/assets/assets/?offset=0&limit=100&search=web' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

If the external system is displayed step by step in a node tree, you can also pull the assets under the node by node:

```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/3728f004-99a2-4fca-9577-84d5ffcf9eff/assets/?offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Record the `id` (UUID) of each asset in the returned result, which will be used when creating authorization later.

### Step 2: Query the applicant’s user ID

After the user submits an application in the external system, the external system uses its user name to check the JumpServer user ID:

```sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Get the user's `id` field (UUID) from the returned list. If the list is empty, it means that the user is not in the current organization or has not been synchronized to JumpServer, and the applicant should be prompted in the external system.

### Step 3: Create a limited time authorization after approval

After the external system is approved, the asset authorization interface is called. Key points:

- `users`: applicant user ID list; `assets`: applied asset ID list;
- `accounts`: Authorized account, you can fill in the account ID, or special values `@ALL` (all accounts), `@SPEC` (specified account), `@INPUT` (manual account), `@USER` (account with the same name);
- `actions`: Authorization actions, optional values `connect`, `upload`, `download`, `copy`, `paste`, `delete`, `share`. According to the principle of least privilege, only `connect` can be given;
- `date_start` / `date_expired`: Authorization validity and expiration time (ISO 8601 format), the authorization will automatically expire after expiration.

```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "name":"apply-20260722-zhangsan-web-server-01",
            "users":["cf7a1f14-0c70-4209-8196-c24cb7ec41a4"],
            "assets":["7e39e2e6-88cb-4a34-a465-fca7b1de1a95"],
            "accounts":["@ALL"],
            "actions":["connect"],
            "is_active":true,
            "date_start":"2026-07-22T09:00:00.000Z",
            "date_expired":"2026-07-29T09:00:00.000Z",
            "comment":"External system tickets TICKET-1001 Automatically created after approval"
        }'
```

Returning `201` indicates that the creation is successful. Record the authorization `id` in the return body. The external system should associate it with the work order to facilitate subsequent early recycling or auditing. It is recommended that `name` include a work order number and other business identifiers to ensure traceability and no duplicate names.

### Step 4: Back-verify that the user has authorized assets

After the creation is successful, you can query the user's currently authorized asset list for back verification, confirm that the target asset has appeared in the results, and then notify the user in the external system that "the authorization has taken effect":

```sh
curl -X GET 'https://localhost/api/v1/perms/users/cf7a1f14-0c70-4209-8196-c24cb7ec41a4/assets/?offset=0&limit=15' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### Step 5: Automatic expiration and early recycling (optional)

The authorization automatically expires after `date_expired` expires (the read-only field `is_expired` of the authorization record becomes `true`, `is_valid` becomes `false`), the user can no longer connect to assets through the authorization, and the external system does not need to call any interface when it expires. If the approval is revoked or needs to be withdrawn in advance, just delete the authorization:

```sh
curl -X DELETE 'https://localhost/api/v1/perms/asset-permissions/b2c45ce6-6b9c-4270-a073-567ff0e0ade8/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

## Complete sample code

The following Python script uses API Key (AK/SK) signature authentication to string together the complete process of "synchronizing the list → checking users → creating limited time authorization → checking back", and can be directly used as a reference implementation for external systems to connect to JumpServer.

```python
# Python Example：External systems apply for assets and automatically authorize them（Complete process）
# Depend on：pip install requests httpsig

import sys
import json
from datetime import datetime, timedelta

import requests
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"                            # JumpServer Address
KEY_ID      = "your id"                                      # API Key ID
KEY_SECRET  = "your secret"                                  # API Key Secret
ORG_ID      = "00000000-0000-0000-0000-000000000002"         # organization ID

# ---- Business input（The application work order actually comes from the external system.）----
APPLY_USERNAME   = "zhangsan"          # The applicant is in JumpServer username in
APPLY_ASSET_NAME = "web-server-01"     # The name of the asset being applied for
APPLY_TICKET_NO  = "TICKET-1001"       # External system ticket number，for traceability
PERMIT_DAYS      = 7                   # Authorization validity period（day），Automatically expires


def build_auth():
    """structure AK/SK of HTTP Signature authentication object"""
    signature_headers = ['(request-target)', 'accept', 'date']
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = signature_headers
    )


def build_headers():
    """Regenerated on every request Date head，The signature depends on this value"""
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }


def api_get(path, params = None):
    url = f"{API_URL}{path}"
    response = requests.get(
        url, auth = build_auth(), headers = build_headers(),
        params = params
    )
    response.raise_for_status()
    return response.json()


def api_post(path, data):
    url = f"{API_URL}{path}"
    response = requests.post(
        url, auth = build_auth(), headers = build_headers(),
        data = json.dumps(data)
    )
    response.raise_for_status()
    return response.json()


def sync_nodes_and_assets():
    """step one：Synchronize node and asset list（External systems can be executed regularly and cached）"""
    nodes = api_get("/api/v1/assets/nodes/", params = {"offset": 0, "limit": 100})
    assets = api_get("/api/v1/assets/assets/", params = {"offset": 0, "limit": 100})
    print(f"Synchronization completed：node {nodes.get('count', 0)} a，assets {assets.get('count', 0)} Taiwan")
    return nodes.get("results", []), assets.get("results", [])


def get_user_by_username(username):
    """Step 2：Check users by username ID"""
    data = api_get("/api/v1/users/users/", params = {"username": username})
    # When no paging parameters are passed, the interface directly returns the list.，Passed offset/limit then return with results paging object
    users = data if isinstance(data, list) else data.get("results", [])
    for user in users:
        if user.get("username") == username:
            return user
    return None


def find_asset_by_name(assets, name):
    """Pinpoint assets by name from a synced asset list"""
    for asset in assets:
        if asset.get("name") == name:
            return asset
    return None


def create_time_limited_permission(user_id, asset_id, days):
    """Step 3：Create a limited-time asset license，Automatically expires"""
    now = datetime.utcnow()
    time_form = "%Y-%m-%dT%H:%M:%S.000Z"
    data = {
        # name Bring the work order number and date，Guaranteed to be unique and traceable
        "name": f"apply-{APPLY_TICKET_NO}-{now.strftime('%Y%m%d%H%M%S')}",
        "users": [user_id],
        "assets": [asset_id],
        "accounts": ["@ALL"],          # It can also be an account id，or @SPEC/@INPUT/@USER
        "actions": ["connect"],        # least privilege：Only allow connections
        "is_active": True,
        "date_start": now.strftime(time_form),
        "date_expired": (now + timedelta(days = days)).strftime(time_form),
        "comment": f"External system tickets {APPLY_TICKET_NO} Automatically created after approval"
    }
    return api_post("/api/v1/perms/asset-permissions/", data)


def verify_user_perm_assets(user_id, asset_id):
    """Step 4：Back-check whether the user’s authorized assets include the target asset"""
    data = api_get(
        f"/api/v1/perms/users/{user_id}/assets/",
        params = {"offset": 0, "limit": 100}
    )
    permed = data.get("results", [])
    return any(asset.get("id") == asset_id for asset in permed)


def main():
    try:
        # 1. Synchronize node and asset list
        _nodes, assets = sync_nodes_and_assets()

        # 2. Counter-check applicant user ID
        user = get_user_by_username(APPLY_USERNAME)
        if not user:
            print(f"Error：User {APPLY_USERNAME} Does not exist in current organization，Please confirm that the user has synced")
            sys.exit(1)
        print(f"Applicant：{user['name']}（id={user['id']}）")

        # 3. Locate the asset being applied for
        asset = find_asset_by_name(assets, APPLY_ASSET_NAME)
        if not asset:
            print(f"Error：assets {APPLY_ASSET_NAME} not found，Please confirm that the assets are managed and organized correctly")
            sys.exit(1)
        print(f"target asset：{asset['name']}（id={asset['id']}）")

        # 4. Create a time-limited license（The external system should perform this step after passing its own approval.）
        perm = create_time_limited_permission(user["id"], asset["id"], PERMIT_DAYS)
        print("Limited time authorization created successfully:")
        print(json.dumps(
            {k: perm.get(k) for k in
             ("id", "name", "date_start", "date_expired", "is_active")},
            indent = 2, ensure_ascii = False
        ))

        # 5. Back-check authorization results
        if verify_user_perm_assets(user["id"], asset["id"]):
            print(f"Passed back-inspection：{APPLY_USERNAME} Already accessible {APPLY_ASSET_NAME}，"
                  f"{PERMIT_DAYS} Tianhou authorization automatically expires")
        else:
            print("Reinspection failed：The authorization has been created but the user has not yet queried the asset.，"
                  "Please check the organization、date_start Whether it is future time, etc.")

    except requests.exceptions.HTTPError as e:
        print(f"API Request failed: {e}")
        if e.response is not None:
            print(f"Response content: {e.response.text}")
        sys.exit(1)
    except requests.exceptions.RequestException as e:
        print(f"network error: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

## FAQ

**Q1: After the authorization expires, does the external system still need to call the interface to cancel the authorization? **

unnecessary. The authorization automatically expires after `date_expired` (the record's read-only field `is_expired` is `true` and `is_valid` is `false`), and users can no longer connect to assets through this authorization. Expired authorization records will be retained for auditing purposes; if you need to clean them up or recycle them in advance, call `DELETE /api/v1/perms/asset-permissions/{id}/` to delete them.

**Q2: Creating authorization returns 400. What is the general reason? **

Common reasons: `name` has the same name as an existing authorization (it is recommended to include a work order number to ensure uniqueness); `date_start` / `date_expired` time format is incorrect (ISO 8601 format should be used, such as `2026-07-22T09:00:00.000Z`) or the expiration time is earlier than the start time; the ID in `users` / `assets` does not exist currently The organization specified by `X-JMS-ORG`. You can check the errors one by one based on the field error prompts in the response body.

**Q3: The authorization was created successfully (201), but the asset cannot be found in the back verification interface? **

Check in sequence: whether the request header `X-JMS-ORG` is consistent with when the authorization was created (the cross-organization query result is empty); whether `date_start` is set to the future time (the authorization has not yet taken effect); whether `is_active` is `true`. In addition, the backtest interface path parameter is the user ID (UUID), and passing it as a user name will result in 404.

**Q4: How should `accounts` and `actions` be filled in? **

`accounts` can be filled with the account ID on the asset, or special values can be used: `@ALL` (all accounts), `@SPEC` (specified account), `@INPUT` (manual account), `@USER` (account with the same name). The optional values ​​of `actions` are `connect`, `upload`, `download`, `copy`, `paste`, `delete`, `share`; for external system application scenarios, it is recommended to only grant `connect` according to the principle of least privilege, and add `upload` / `download` if there are file transfer requirements.
