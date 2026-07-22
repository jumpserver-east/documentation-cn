# Practical case: User life cycle automation (interfacing with HR/OA)

## Scene description

Enterprises usually maintain the entry and exit status of employees in the HR/OA system. If bastion host accounts are manually opened and recycled, problems such as "slow onboarding and forgotten to reclaim after leaving" may occur. The remaining access rights of resigned employees are even more serious security risks. This case demonstrates how to link with the HR system through the JumpServer API: automatically create bastion host users when employees join the company, join user groups by department, and inherit department authorizations; automatically disable accounts when employees leave, move out of user groups, and reclaim personal direct authorizations. After an observation period to confirm that there are no remaining problems, the accounts will be deleted. The HR system only needs to trigger this script through Webhook or scheduled tasks at the onboarding/offboarding process node.

## Preconditions

- An API credential with administrative rights: the Bearer Token of the organization administrator (or system administrator) user, or the user's API Key (AK/SK, with `httpsig` signature authentication), choose one of the two.
- The organization ID of the known target organization (request header `X-JMS-ORG`, defaults to `Default` organization `00000000-0000-0000-0000-000000000002`).
- The user groups corresponding to the departments (such as "operations Group") and the assets/nodes that the departments need to access have been planned in JumpServer.
- It is recommended to adopt the "authorization binding user group" mode: the object of authorization rules is department user groups instead of individuals. In this way, you will be authorized when you join the team, and you will recover the authorization when you leave the team, so as to avoid fragmentation of authorization.
- Running environment: Python 3, install `requests` (if you use AK/SK authentication, install `httpsig`).

All requests carry the following request headers:

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/users/groups/` | Query user groups by name and obtain department group IDs |
| POST | `/api/v1/users/users/` | Create a user (open an account after joining the company, and specify a user group at the same time) |
| GET | `/api/v1/users/groups/{id}/` | Query user group details (retrieve existing members for merging) |
| PATCH | `/api/v1/users/groups/{id}/` | Update the user group `users` field (add existing users to the user group) |
| POST | `/api/v1/perms/asset-permissions/` | Create asset authorization (authorization by department user group, first execution) |
| GET | `/api/v1/users/users/` | Query the user by `username` and locate the account of the resigned employee |
| PATCH | `/api/v1/users/users/{id}/` | Disable the user (`is_active=false`) and clear the user group |
| GET | `/api/v1/perms/asset-permissions/` | Press `user_id` to query the direct authorization under the user name |
| GET / PATCH | `/api/v1/perms/asset-permissions/{id}/` | Query authorization details and remove the user from `users` |
| DELETE | `/api/v1/users/users/{id}/` | Delete the user (executed after the termination observation period) |

> Field-level details for each endpoint are available in your environment's online documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

### Onboarding process

#### Step 1: Query the department user group and obtain the group ID

Query the corresponding user group (such as "operations Group") according to the department name on the HR side, and get the group ID for subsequent use when creating users.

```sh
curl -X GET 'https://localhost/api/v1/users/groups/?name=%E8%BF%90%E7%BB%B4%E7%BB%84' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Get the `id` field from the returned list, for example `7413d36d-cf37-45f4-b42a-cd5eeff1f4ff`.

#### Step 2: Create users and join user groups directly

When creating a user, directly specify the user group to which it belongs through the `groups` field, and complete "opening an account + joining the group" in one step. `password_strategy` is recommended to use `email`, and the system will send the password setting link to the employee's mailbox to avoid clear text passwords in the script; the HR employee number can be written back in `comment` to facilitate reconciliation.

```sh
curl -X POST 'https://localhost/api/v1/users/users/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "Zhang San",
        "username": "zhangsan",
        "email": "zhangsan@example.com",
        "password_strategy": "email",
        "source": "local",
        "groups": ["7413d36d-cf37-45f4-b42a-cd5eeff1f4ff"],
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}],
        "date_expired": "2096-12-31T00:00:00.000Z",
        "comment": "HR Job number E10086，Onboarding process automatically created"
    }'
```

If the creation is successful, the complete user object will be returned, and the `id` (user UUID) will be recorded.

#### Step 3 (optional): Add existing users to user groups

If the account already exists (such as job transfer scenario), use PATCH to update the `users` field of the user group instead. Note that `users` is a full replacement. You must first get the group details to retrieve the existing member IDs, merge the new user IDs into them, and then submit. Otherwise, other members will be removed from the group.

```sh
# Query group details first，Get existing members
curl -X GET 'https://localhost/api/v1/users/groups/7413d36d-cf37-45f4-b42a-cd5eeff1f4ff/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# reuse「Existing members + new member」Submit full amount
curl -X PATCH 'https://localhost/api/v1/users/groups/7413d36d-cf37-45f4-b42a-cd5eeff1f4ff/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "users": [
            "3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001",
            "5e2d8c0a-1b7f-4a2e-8d3c-9f1a2b3c0002"
        ]
    }'
```

#### Step 4: Create asset authorizations for department user groups (only required for the first time per department)

The authorization object points to the user group (`user_groups`), and the asset scope is specified with nodes (`nodes`) or assets (`assets`). This authorization can be created once and will be automatically inherited when new employees join the group. There is no need to create it again for everyone.

```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "ops_group_linux_perm",
        "user_groups": ["7413d36d-cf37-45f4-b42a-cd5eeff1f4ff"],
        "nodes": ["1a2b3c4d-0000-0000-0000-00000000abcd"],
        "assets": [],
        "accounts": ["@SPEC", "root"],
        "protocols": ["all"],
        "actions": ["connect", "upload", "download"],
        "is_active": true,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2096-12-31T00:00:00.000Z",
        "comment": "operations group Linux Server authorization，Onboarding automation process creation"
    }'
```

`@SPEC` in `accounts` represents the specified account (used with the specific account name), and also supports special values such as `@ALL`; the optional values ​​of `actions` are `connect`, `upload`, `download`, `copy`, `paste`, `delete`, `share`.

### Resignation process

#### Step 5: Locate the resigned employee’s account by user name

The HR system usually gives the employee number or user name. First press `username` to query accurately to get the user UUID.

```sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

#### Step 6: Disable account and remove from all user groups (observation period begins)

After disabling (`is_active=false`), the user will be unable to log in immediately; at the same time, if `groups` is left blank, the user will immediately lose all authorizations from the user group. Do not delete it directly - keep the account for an observation period (such as 30 days) to facilitate audit traceability and misoperation rollback.

```sh
curl -X PATCH 'https://localhost/api/v1/users/users/3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ "is_active": false, "groups": [] }'
```

#### Step 7: Reclaim direct authorization in the individual’s name

In addition to group authorizations, users may have authorizations that are directly bound to individuals (such as authorizations generated by work order approval). First list the relevant authorizations by `user_id`, then query the details one by one, remove the user from `users` and then PATCH back (the same is full replacement).

```sh
# List the authorizations associated with this user
curl -X GET 'https://localhost/api/v1/perms/asset-permissions/?user_id=3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# Authorize each item：After checking the details，Remove the user's users Submit the full list
curl -X PATCH 'https://localhost/api/v1/perms/asset-permissions/9c8d7e6f-0000-0000-0000-0000000000aa/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ "users": ["5e2d8c0a-1b7f-4a2e-8d3c-9f1a2b3c0002"] }'
```

> Note: Results filtered by `user_id` include authorizations associated via user groups. The script only needs to process the authorizations that do contain the user ID in `users`; the group authorization has been automatically recycled when the user group is removed in step 6.

#### Step 8: Delete the user after the observation period is over

After confirming that there are no abnormalities during the observation period (no audit traceability requirements, no incorrect operation complaints), delete the account to complete the closed loop. Successful deletion returns `204 No Content`.

```sh
curl -X DELETE 'https://localhost/api/v1/users/users/3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

## Complete sample code

```python
# -*- coding: utf-8 -*-
"""
JumpServer User lifecycle automation script（docking HR/OA）

Onboarding onboard : Query department user groups -> Create users and join groups -> （first time）Create asset entitlements for user groups
Resign offboard: Target users -> Disable and remove all user groups -> Recycling individual direct authorization
Delete purge   : Delete users after the termination observation period

Authentication method：Bearer Token（If required AK/SK，Can be used instead httpsig of HTTPSignatureAuth，
Reference《User management》Chapter Python Example）
"""

import sys
import requests

API_URL = "https://localhost"                                # JumpServer Address
TOKEN   = "b96810faac725563304dada8c323c4fa061863d4"         # Administrator Token
ORG_ID  = "00000000-0000-0000-0000-000000000002"             # organization ID

session = requests.Session()
session.headers.update({
    "Authorization": f"Bearer {TOKEN}",
    "X-JMS-ORG": ORG_ID,
    "Content-Type": "application/json",
})
session.verify = False   # Self-signed certificate environment；In the production environment, it is recommended to configure a trusted certificate and change it to True


def check(resp, action):
    """Unified error handling：Not 2xx Throw an exception directly，204 unresponsive body"""
    if not resp.ok:
        raise RuntimeError(f"{action} failed: HTTP {resp.status_code} {resp.text}")
    return resp.json() if resp.status_code != 204 else None


def as_results(data):
    """Compatible with paging（{'results': [...]}）vs. non-pagination（[...]）Two return structures"""
    return data.get("results", []) if isinstance(data, dict) else data


# ---------- Onboarding ----------

def get_group_id(group_name):
    """Query user group by name，Return to group ID"""
    resp = session.get(f"{API_URL}/api/v1/users/groups/",
                       params={"name": group_name})
    groups = as_results(check(resp, f"Query user group {group_name}"))
    if not groups:
        raise RuntimeError(f"User group does not exist: {group_name}，Please create it in the console first")
    return groups[0]["id"]


def create_user(name, username, email, group_id, emp_no):
    """Create users and directly join department user groups，Passwords are set by employees via email link"""
    payload = {
        "name": name,
        "username": username,
        "email": email,
        "password_strategy": "email",
        "source": "local",
        "groups": [group_id],
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}],
        "comment": f"HR Job number {emp_no}，Onboarding process automatically created",
    }
    resp = session.post(f"{API_URL}/api/v1/users/users/", json=payload)
    user = check(resp, f"Create user {username}")
    print(f"[Onboarding] The user has been created and joined the user group: {username} ({user['id']})")
    return user


def ensure_group_permission(group_id, perm_name, node_ids, accounts):
    """Make sure the department user group has asset authorization；Create if it does not exist（Only once per department）"""
    resp = session.get(f"{API_URL}/api/v1/perms/asset-permissions/",
                       params={"name": perm_name})
    if as_results(check(resp, "Query authorization")):
        print(f"[Onboarding] Authorization already exists，Skip creation: {perm_name}")
        return
    payload = {
        "name": perm_name,
        "user_groups": [group_id],
        "nodes": node_ids,
        "assets": [],
        "accounts": accounts,                      # Such as ["@SPEC", "root"]
        "protocols": ["all"],
        "actions": ["connect", "upload", "download"],
        "is_active": True,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2096-12-31T00:00:00.000Z",
        "comment": "Department authorization for onboarding automation process creation",
    }
    resp = session.post(f"{API_URL}/api/v1/perms/asset-permissions/",
                        json=payload)
    perm = check(resp, f"Create authorization {perm_name}")
    print(f"[Onboarding] Department authorization has been created: {perm_name} ({perm['id']})")


def onboard(name, username, email, dept_group, emp_no,
            node_ids, accounts):
    """Entrance：HR The system calls at the onboarding node"""
    group_id = get_group_id(dept_group)
    user = create_user(name, username, email, group_id, emp_no)
    ensure_group_permission(group_id, f"{dept_group}_auto_perm",
                            node_ids, accounts)
    return user


# ---------- Resign ----------

def find_user_by_username(username):
    """Pinpoint users by username，Return user object"""
    resp = session.get(f"{API_URL}/api/v1/users/users/",
                       params={"username": username})
    users = as_results(check(resp, f"Query user {username}"))
    matched = [u for u in users if u["username"] == username]
    if not matched:
        raise RuntimeError(f"User not found: {username}")
    return matched[0]


def disable_user(user_id):
    """Disable the account and remove all user groups：Immediately lose the ability to log in and all group authorizations"""
    resp = session.patch(f"{API_URL}/api/v1/users/users/{user_id}/",
                         json={"is_active": False, "groups": []})
    check(resp, "Disable user")
    print(f"[Resign] User disabled and removed from all user groups: {user_id}")


def revoke_direct_permissions(user_id):
    """Recycling authorizations tied directly to an individual（users Replace all fields，Write back after deleting the user）"""
    resp = session.get(f"{API_URL}/api/v1/perms/asset-permissions/",
                       params={"user_id": user_id})
    perms = as_results(check(resp, "Query user associated authorization"))
    for perm in perms:
        detail_url = f"{API_URL}/api/v1/perms/asset-permissions/{perm['id']}/"
        detail = check(session.get(detail_url), "Query authorization details")
        old_users = detail.get("users") or []
        remain = [u["id"] for u in old_users if u["id"] != user_id]
        if len(remain) == len(old_users):
            continue   # The authorization is associated via the user group，users There is no such person in，skip
        resp = session.patch(detail_url, json={"users": remain})
        check(resp, f"Recycling authorization {detail['name']}")
        print(f"[Resign] User removed from authorization: {detail['name']}")


def offboard(username):
    """Resignation entrance：HR The system calls it at the resignation node。Disable observation first，Do not delete immediately"""
    user = find_user_by_username(username)
    disable_user(user["id"])
    revoke_direct_permissions(user["id"])
    print(f"[Resign] {username} Processing completed，Entering the observation period，Execute after expiration purge Delete")
    return user["id"]


def purge(username):
    """Delete users after the observation period ends，Operation is not recoverable"""
    user = find_user_by_username(username)
    if user.get("is_active"):
        raise RuntimeError(f"User {username} Still enabled，"
                           f"Please perform the resignation disabling process first，Confirm before deleting")
    resp = session.delete(f"{API_URL}/api/v1/users/users/{user['id']}/")
    check(resp, f"Delete user {username}")
    print(f"[Delete] User has been deleted: {username}")


if __name__ == "__main__":
    # Usage:
    #   python user_lifecycle.py onboard  Zhang San zhangsan zhangsan@example.com operations group E10086
    #   python user_lifecycle.py offboard zhangsan
    #   python user_lifecycle.py purge    zhangsan
    try:
        action = sys.argv[1]
        if action == "onboard":
            name, username, email, dept, emp_no = sys.argv[2:7]
            onboard(name, username, email, dept, emp_no,
                    node_ids=["1a2b3c4d-0000-0000-0000-00000000abcd"],
                    accounts=["@SPEC", "root"])
        elif action == "offboard":
            offboard(sys.argv[2])
        elif action == "purge":
            purge(sys.argv[2])
        else:
            print("Unknown operation，support: onboard / offboard / purge")
    except (IndexError, ValueError):
        print("Not enough parameters，Please refer to the usage in the script header comments.")
    except RuntimeError as e:
        print(f"Execution failed: {e}")
        sys.exit(1)
```

## FAQ

**Q1: After adding a user to a user group, other members suddenly "disappeared"? **
A: The `users` field of `PATCH /api/v1/users/groups/{id}/` and the `groups` field of `PATCH /api/v1/users/users/{id}/` are fully replaced instead of appended. Before adding members, you must first GET the existing list, merge it and then submit it; the script in this article directly uses the `groups` field to join the group when creating a user, which naturally avoids this pitfall.

**Q2: Why is it recommended to bind authorization to user groups instead of directly authorizing it to individuals? **
A: After the authorization is bound to the user group, it will take effect when joining the group and will be recycled when leaving the group. The number of authorization rules is constant and easy to audit. If authorization is granted on a person-by-person basis, employee flow will cause authorization fragmentation, and it is easy to miss collection when leaving the company. Direct personal authorization should only be reserved for temporary scenarios such as work orders, and be recycled uniformly during the offboarding process (see step 7).

**Q3: Calling the interface returns 403 or the newly created resource cannot be found? **
A: First check whether the user corresponding to the token has the role of organization administrator/system administrator; then check whether `X-JMS-ORG` is consistent with the organization to which the target resource belongs - user groups and authorizations are organization-level resources. If the organization ID does not match, it will appear as 404 or an empty list. When creating a user, `system_roles` and `org_roles` need to be filled in according to the actual role ID of the environment.

**Q4: Why not DELETE directly when resigning, but disable the observation period first? **
A: Deletion is irreversible and will cut off the authorization association with the user, which is not conducive to audit traceability and rollback of misoperations (such as HR status synchronization errors). Recommended process: First disable and recycle the authorization with `is_active=false` (the user cannot log in immediately, the risk has been eliminated), keep it for an observation period of about 30 days, and then execute DELETE after confirming that there are no abnormalities.

**Q5: After being disabled, will the online session already opened by the user be automatically disconnected? **
A: Disabling blocks subsequent logins. To be on the safe side, after the resignation is processed, you should check whether the user has an online session in the "Session Management" of the console and terminate it manually.
