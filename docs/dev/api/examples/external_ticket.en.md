# Practical case: docking with external work order system

## Scene description

Enterprises have usually built a unified ITSM/OA approval platform (such as Jira, ServiceNow, DingTalk approval, etc.), and operations personnel hope to connect JumpServer's asset application and authorization process with the existing approval system to avoid "approving both sides again." This case covers two typical docking directions:

- **Direction 1**: The external system submits the asset application work order to JumpServer on behalf of the user, and then polls the work order status; the approval action is still completed by the approver in JumpServer, and the external system is only responsible for initiating and tracking.
- **Direction 2**: The approval process is completely executed in the external system; after the external approval is passed, the docking program calls the JumpServer interface to approve existing work orders, or directly creates asset authorization to achieve "external approval and JumpServer implementation".

## Preconditions

- JumpServer has configured a work order approval process of type `apply_asset` (asset application), which can be confirmed by `GET /api/v1/tickets/flows/?type=apply_asset` (it can also be configured on the page Work Order - Process Settings).
- The docking program has obtained the authentication credentials of the corresponding identity (Bearer Token or API Key AK/SK):
    - Submit a work order: It is recommended to use the credentials of the applicant. The identity of the work order applicant (applicant) shall prevail when calling the interface;
    - Approval/rejection of a work order: The credentials of the assignee of the current approval step of the work order must be used;
    - Direct creation of asset authorization: Credentials of an account with asset authorization management rights (such as organization administrator) are required.
- Known target organization ID (request header `X-JMS-ORG`), as well as pre-data such as asset ID, node ID, user ID to be applied for/authorized.
- External systems (or docking middleware) on the network can access the HTTPS interface of JumpServer.

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/tickets/flows/` | Query the work order approval process configuration (confirm that the apply_asset process already exists) |
| POST | `/api/v1/tickets/apply-asset-tickets/open/` | Submit (open) asset application work order |
| GET | `/api/v1/tickets/tickets/` | Query the work order list by status/type and other conditions |
| GET | `/api/v1/tickets/tickets/{id}/` | Query basic information of a single work order (for polling status) |
| GET | `/api/v1/tickets/apply-asset-tickets/{id}/` | Query asset application work order details |
| PATCH | `/api/v1/tickets/apply-asset-tickets/{id}/approve/` | Approval of asset application work orders |
| PUT | `/api/v1/tickets/apply-asset-tickets/{id}/reject/` | Reject asset request work order |
| PUT | `/api/v1/tickets/apply-asset-tickets/{id}/close/` | Close (cancel) asset application work order |
| POST | `/api/v1/perms/asset-permissions/` | Create asset authorization directly (without going through a work order) |

> Field-level details for each endpoint are available in the JumpServer online API documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

### Direction 1: The external system submits work orders on behalf of users and polls the status

#### Step 1: Confirm that the approval process is configured

Before submitting an asset application work order, make sure that the current organization has configured an approval process of the `apply_asset` type, otherwise the submission will fail.

```sh
curl -X GET 'https://localhost/api/v1/tickets/flows/?type=apply_asset&offset=0&limit=15' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

If the returned list is not empty, it means that the process has been configured, focusing on `approval_level` (approval level) and `rules` (acceptees at all levels).

#### Step 2: Submit an asset application work order on behalf of the user

The external system collects the application information (assets, account, action, validity period) in its own form, and then calls the open work order interface. Note that the Token or AK/SK of the **applicant user** is used for the call. The work order applicant is the user.

```sh
curl -X POST 'https://localhost/api/v1/tickets/apply-asset-tickets/open/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title": "ITSM-2026072201 Request access to production Web server",
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["b4f205af-4353-49ef-befa-ff9095d52a27"],
        "apply_accounts": ["@ALL"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T00:00:00.000Z",
        "apply_date_expired": "2026-08-22T00:00:00.000Z",
        "comment": "Corresponding external work order number ITSM-2026072201"
    }'
```

It is recommended to write the external work order number into `title` or `comment` to facilitate two-way reconciliation. The `id` in the return body is the JumpServer work order ID, and the external system should save it in association with its own work order number.

#### Step 3: Poll for ticket status

Use the work order ID returned in the previous step to periodically poll the status. `state.value` is the approval action: `pending` (pending), `approved` (agreed), `rejected` (rejected); `status.value` is the work order status: `open` (in progress), `closed` (ended).

```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

If you need batch reconciliation, you can also query the work order list according to conditions:

```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/?type=apply_asset&status=open&state=pending' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

After polling to `state.value` is `approved` or `rejected`, the external system can write back the results and notify the user. The approver can process it normally on the JumpServer page (Work Order - To-Do Work Order) without being aware of the external system.

#### Step 4 (optional): The user closes the ticket when the external system cancels the order

If the user cancels the application in the external system, the corresponding work order on the JumpServer side can be closed synchronously (called using the applicant identity).

```sh
curl -X PUT 'https://localhost/api/v1/tickets/apply-asset-tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/close/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title": "ITSM-2026072201 Request access to production Web server",
        "org_id": "00000000-0000-0000-0000-000000000002"
    }'
```

### Direction 2: Put the approval in the external system, and write it back to JumpServer after passing

After external approval is passed, there are two ways to implement the docking program, choose one according to your needs.

#### Method A: Approve existing JumpServer tickets

Applicable to the scenario where "the user is still placing the bill of lading on JumpServer (or generating the bill of lading by direction), but the approval conclusion is subject to the external system". After the external approval is passed, the docking program uses the credentials of the assignee of the current approval step of the work order to call the approval interface; the approval is reflected in the work order record of JumpServer, and the authorization is automatically generated by JumpServer based on the content of the work order.

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
        "apply_date_start": "2026-07-22T00:00:00.000Z",
        "apply_date_expired": "2026-08-22T00:00:00.000Z"
    }'
```

The approver can modify the assets, accounts, actions, and validity period in the request body, which constitutes approval with changes. When the external system rejects a ticket, call the rejection endpoint:

```sh
curl -X PUT 'https://localhost/api/v1/tickets/apply-asset-tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/reject/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title": "ITSM-2026072201 Request access to production Web server",
        "org_id": "00000000-0000-0000-0000-000000000002",
        "comment": "External work order ITSM-2026072201 Approval failed"
    }'
```

#### Method B: Create asset authorization directly after external approval is passed

Applicable to the scenario of "no work orders at all in JumpServer". After the external approval is passed, the docking program uses the administrator credentials to directly create asset authorization, and the user immediately gains access rights.

```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "ITSM-2026072201-zhangsan-web01",
        "users": ["8f6e2b1a-3c4d-4e5f-9a0b-1c2d3e4f5a6b"],
        "assets": ["b4f205af-4353-49ef-befa-ff9095d52a27"],
        "accounts": ["@ALL"],
        "actions": ["connect", "upload", "download"],
        "is_active": true,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2026-08-22T00:00:00.000Z",
        "comment": "Corresponding external work order number ITSM-2026072201"
    }'
```

It is recommended to bring the external work order number for the authorization name `name` to facilitate expiration cleanup and audit traceability. After expiration, the authorization automatically expires (`date_expired`), and can also be actively recycled by the docking program by calling the delete authorization interface.

## Complete sample code

The following Python script strings together the complete process: confirming process configuration, picking up orders on behalf of users, polling status; and providing functions for writing back external approval results (approving/rejecting existing work orders, directly creating authorizations). Tokens for the three identities are configured on demand.

```python
# -*- coding: utf-8 -*-
"""
JumpServer Example of docking with external work order system
Direction one：Submit asset application work orders on behalf of users + Polling work order status
Direction two：Approval of existing work orders after passing external approval，Or create asset authorization directly
Depend on：pip install requests
"""

import json
import time

import requests

API_URL = "https://localhost"
ORG_ID = "00000000-0000-0000-0000-000000000002"

# three identities Token：Applicant（bill of lading）、approver（Approve existing work orders）、Administrator（Create authorization directly）
APPLICANT_TOKEN = "your applicant token"
APPROVER_TOKEN = "your approver token"
ADMIN_TOKEN = "your admin token"

VERIFY_SSL = False   # The self-signed certificate environment is set to False，In the production environment, it is recommended to configure a trusted certificate and set it to True

ASSET_ID = "b4f205af-4353-49ef-befa-ff9095d52a27"   # Assets applied for ID
USER_ID = "8f6e2b1a-3c4d-4e5f-9a0b-1c2d3e4f5a6b"    # User during direct authorization ID
EXTERNAL_TICKET_NO = "ITSM-2026072201"               # External system ticket number


def build_headers(token):
    """Construct a request header with authentication information"""
    return {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {token}",
        "X-JMS-ORG": ORG_ID,
    }


def request_api(method, path, token, **kwargs):
    """Unified request entry，Contains error handling"""
    url = f"{API_URL}{path}"
    try:
        response = requests.request(
            method, url, headers=build_headers(token),
            verify=VERIFY_SSL, timeout=30, **kwargs
        )
        response.raise_for_status()
        if response.status_code == 204:
            return None
        return response.json()
    except requests.exceptions.HTTPError as e:
        # Print error details returned by the server，Convenient to locate parameter problems
        print(f"HTTP Error: {e}, Response content: {e.response.text}")
        raise
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")
        raise


def check_apply_asset_flow():
    """steps 1：Confirm apply_asset Approval process configured"""
    result = request_api(
        "GET", "/api/v1/tickets/flows/", APPLICANT_TOKEN,
        params={"type": "apply_asset", "offset": 0, "limit": 15},
    )
    flows = result.get("results", result) if isinstance(result, dict) else result
    if not flows:
        raise RuntimeError("Not configured apply_asset Approval process，please first JumpServer Configure the work order process in")
    print(f"Approval process configured，total {len(flows)} Article")


def submit_asset_ticket():
    """steps 2：Submit asset application work orders on behalf of users（Use applicant identity）"""
    data = {
        "title": f"{EXTERNAL_TICKET_NO} Request access to production Web server",
        "org_id": ORG_ID,
        "apply_assets": [ASSET_ID],
        "apply_accounts": ["@ALL"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T00:00:00.000Z",
        "apply_date_expired": "2026-08-22T00:00:00.000Z",
        "comment": f"Corresponding external work order number {EXTERNAL_TICKET_NO}",
    }
    ticket = request_api(
        "POST", "/api/v1/tickets/apply-asset-tickets/open/",
        APPLICANT_TOKEN, data=json.dumps(data),
    )
    print(f"Work order submitted successfully, id={ticket['id']}, serial_num={ticket.get('serial_num')}")
    return ticket["id"]


def poll_ticket(ticket_id, interval=30, max_rounds=120):
    """steps 3：Polling work order status，Until approval is completed or times out"""
    for _ in range(max_rounds):
        ticket = request_api(
            "GET", f"/api/v1/tickets/tickets/{ticket_id}/", APPLICANT_TOKEN,
        )
        state = ticket["state"]["value"]      # pending / approved / rejected
        status = ticket["status"]["value"]    # open / closed
        print(f"Current status: state={state}, status={status}")
        if state != "pending":
            return state
        time.sleep(interval)
    raise TimeoutError("Poll timeout，The work order is still pending approval")


def approve_ticket(ticket_id):
    """Direction two/way A：After external approval is passed，Approve existing work orders（Use approver identity）"""
    data = {
        "org_id": ORG_ID,
        "apply_assets": [ASSET_ID],
        "apply_accounts": ["@ALL"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T00:00:00.000Z",
        "apply_date_expired": "2026-08-22T00:00:00.000Z",
    }
    request_api(
        "PATCH", f"/api/v1/tickets/apply-asset-tickets/{ticket_id}/approve/",
        APPROVER_TOKEN, data=json.dumps(data),
    )
    print(f"work order {ticket_id} Approved")


def reject_ticket(ticket_id):
    """Direction two/way A：After external approval is rejected，Dismiss existing work order（Use approver identity）"""
    data = {
        "title": f"{EXTERNAL_TICKET_NO} Request access to production Web server",
        "org_id": ORG_ID,
        "comment": f"External work order {EXTERNAL_TICKET_NO} Approval failed",
    }
    request_api(
        "PUT", f"/api/v1/tickets/apply-asset-tickets/{ticket_id}/reject/",
        APPROVER_TOKEN, data=json.dumps(data),
    )
    print(f"work order {ticket_id} Dismissed")


def create_asset_permission():
    """Direction two/way B：After external approval is passed，Create asset authorization directly（Use as administrator）"""
    data = {
        "name": f"{EXTERNAL_TICKET_NO}-perm",
        "users": [USER_ID],
        "assets": [ASSET_ID],
        "accounts": ["@ALL"],
        "actions": ["connect", "upload", "download"],
        "is_active": True,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2026-08-22T00:00:00.000Z",
        "comment": f"Corresponding external work order number {EXTERNAL_TICKET_NO}",
    }
    perm = request_api(
        "POST", "/api/v1/perms/asset-permissions/",
        ADMIN_TOKEN, data=json.dumps(data),
    )
    print(f"Asset authorization created successfully, id={perm['id']}")
    return perm["id"]


def main():
    # ---- Direction one：Pick up orders on behalf of users and poll them ----
    check_apply_asset_flow()
    ticket_id = submit_asset_ticket()
    final_state = poll_ticket(ticket_id, interval=30)
    print(f"Work order final status: {final_state}")

    # ---- Direction two：Write back external approval results（Choose one according to the actual docking scenario）----
    # way A：External approval passed -> Approve existing work orders；rejected -> Dismiss
    # approve_ticket(ticket_id)
    # reject_ticket(ticket_id)

    # way B：Not leaving JumpServer work order，Create authorization directly
    # create_asset_permission()


if __name__ == "__main__":
    main()
```

## FAQ

**Q1: The work order submission interface returns 400, prompting an error related to the approval process? **

A: The current organization has not configured the `apply_asset` type of work order approval process. Please configure the process on the JumpServer page (Ticket - Process Settings) or through the `/api/v1/tickets/flows/` interface before placing a ticket. Note that the process is isolated by organization, and `X-JMS-ORG` must be consistent with the bill of lading organization.

**Q2: The applicant for the work order displays the service account of the docking program instead of the actual user? **

A: The work order applicant shall be subject to the authentication identity of the calling interface. In order for the work order to reflect the real applicant user, the external system needs to use the user's own Bearer Token or API Key (AK/SK) to call the bill of lading interface, instead of using a service account to issue the bill on behalf of the user. API Keys can be created in JumpServer for each user and hosted in the docking program.

**Q3: Calling the approve interface returns 403 No permission? **

A: The approval interface must be called by the identity of the assignee of the current approval step of the work order. Ordinary service accounts may not be in the assignee list even if they are organization administrators. Please check the configuration of assignees at each level in the process `rules` and ensure that the docking program uses the credentials of the corresponding assignee; in multi-level approval, each level needs to be called once by the assignee of that level.

**Q4: If `apply_actions` / `actions` passes `all`, an error is reported. What are the values? **

A: The action enumeration values are `connect`, `upload`, `download`, `copy`, `paste`, `delete`, `share`. Aggregate values such as `all` are not accepted. All actions are listed one by one when required. Also note that the time fields (`apply_date_start`, `date_expired`, etc.) are in ISO 8601 datetime format, `apply_date_expired` must be later than `apply_date_start`, otherwise 400 will be returned.
