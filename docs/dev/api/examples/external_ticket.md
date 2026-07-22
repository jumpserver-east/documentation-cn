# 实战案例：对接外部工单系统

## 场景说明

企业通常已经建设了统一的 ITSM/OA 审批平台（如 Jira、ServiceNow、钉钉审批等），运维人员希望将 JumpServer 的资产申请与授权流程和现有审批体系打通，避免"两边各审一遍"。本案例覆盖两个典型对接方向：

- **方向一**：外部系统代用户在 JumpServer 提交资产申请工单，随后轮询工单状态；审批动作仍由审批人在 JumpServer 中完成，外部系统只负责发起与跟踪。
- **方向二**：审批流程完全放在外部系统中执行；外部审批通过后，由对接程序调用 JumpServer 接口审批既有工单，或直接创建资产授权，实现"外部审批、JumpServer 落地"。

## 前置条件

- JumpServer 已配置 `apply_asset`（资产申请）类型的工单审批流程，可通过 `GET /api/v1/tickets/flows/?type=apply_asset` 确认（也可在页面 工单 - 流程设置 中配置）。
- 对接程序已获取相应身份的认证凭证（Bearer Token 或 API Key AK/SK）：
    - 提交工单：建议使用**申请用户本人**的凭证，工单申请人（applicant）以调用接口的身份为准；
    - 审批/驳回工单：必须使用该工单**当前审批步骤受理人**的凭证；
    - 直接创建资产授权：需要具有资产授权管理权限的账号（如组织管理员）的凭证。
- 已知目标组织 ID（请求头 `X-JMS-ORG`），以及要申请/授权的资产 ID、节点 ID、用户 ID 等前置数据。
- 网络上外部系统（或对接中间件）可以访问 JumpServer 的 HTTPS 接口。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/tickets/flows/` | 查询工单审批流程配置（确认 apply_asset 流程已存在） |
| POST | `/api/v1/tickets/apply-asset-tickets/open/` | 提交（打开）资产申请工单 |
| GET | `/api/v1/tickets/tickets/` | 按状态/类型等条件查询工单列表 |
| GET | `/api/v1/tickets/tickets/{id}/` | 查询单个工单基本信息（轮询状态用） |
| GET | `/api/v1/tickets/apply-asset-tickets/{id}/` | 查询资产申请工单详情 |
| PATCH | `/api/v1/tickets/apply-asset-tickets/{id}/approve/` | 审批通过资产申请工单 |
| PUT | `/api/v1/tickets/apply-asset-tickets/{id}/reject/` | 驳回资产申请工单 |
| PUT | `/api/v1/tickets/apply-asset-tickets/{id}/close/` | 关闭（撤销）资产申请工单 |
| POST | `/api/v1/perms/asset-permissions/` | 直接创建资产授权（不经工单） |

> 各接口的字段级明细可在 JumpServer 在线接口文档 `https://<JumpServer地址>/api/docs` 中查看。

## 操作流程

### 方向一：外部系统代用户提交工单并轮询状态

#### 步骤 1：确认审批流程已配置

提交资产申请工单前，先确认当前组织已配置 `apply_asset` 类型的审批流程，否则提交会失败。

```sh
curl -X GET 'https://localhost/api/v1/tickets/flows/?type=apply_asset&offset=0&limit=15' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回列表非空即表示流程已配置，重点关注 `approval_level`（审批级数）与 `rules`（各级受理人）。

#### 步骤 2：代用户提交资产申请工单

外部系统在自己的表单中收集申请信息（资产、账号、动作、有效期），然后调用打开工单接口。注意使用**申请用户本人**的 Token 或 AK/SK 调用，工单申请人即为该用户。

```sh
curl -X POST 'https://localhost/api/v1/tickets/apply-asset-tickets/open/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title": "ITSM-2026072201 申请访问生产 Web 服务器",
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["b4f205af-4353-49ef-befa-ff9095d52a27"],
        "apply_accounts": ["@ALL"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T00:00:00.000Z",
        "apply_date_expired": "2026-08-22T00:00:00.000Z",
        "comment": "对应外部工单号 ITSM-2026072201"
    }'
```

建议把外部工单号写入 `title` 或 `comment`，便于双向对账。返回体中的 `id` 即 JumpServer 工单 ID，外部系统应将其与自身工单号关联保存。

#### 步骤 3：轮询工单状态

用上一步返回的工单 ID 定时轮询状态。`state.value` 为审批动作：`pending`（待处理）、`approved`（已同意）、`rejected`（已拒绝）；`status.value` 为工单状态：`open`（进行中）、`closed`（已结束）。

```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

如需批量对账，也可以按条件查询工单列表：

```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/?type=apply_asset&status=open&state=pending' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

轮询到 `state.value` 为 `approved` 或 `rejected` 后，外部系统即可回写结果、通知用户。审批人在 JumpServer 页面（工单 - 待办工单）正常处理即可，无需感知外部系统。

#### 步骤 4（可选）：用户在外部系统撤单时关闭工单

如果用户在外部系统撤销了申请，可同步关闭 JumpServer 侧对应工单（使用申请人身份调用）。

```sh
curl -X PUT 'https://localhost/api/v1/tickets/apply-asset-tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/close/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title": "ITSM-2026072201 申请访问生产 Web 服务器",
        "org_id": "00000000-0000-0000-0000-000000000002"
    }'
```

### 方向二：审批放在外部系统，通过后回写 JumpServer

外部审批通过后，对接程序有两种落地方式，按需选择其一。

#### 方式 A：审批既有的 JumpServer 工单

适用于"用户仍在 JumpServer 提单（或按方向一代提单），但审批结论以外部系统为准"的场景。外部审批通过后，对接程序使用**工单当前审批步骤受理人**的凭证调用审批接口；审批体现在 JumpServer 的工单记录中，授权由 JumpServer 按工单内容自动生成。

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

审批人可以在请求体中修改资产、账号、动作与有效期，即"改单审批"。外部审批被拒绝时，调用驳回接口：

```sh
curl -X PUT 'https://localhost/api/v1/tickets/apply-asset-tickets/41b36621-dd4d-492e-a72c-be20b2daeea8/reject/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title": "ITSM-2026072201 申请访问生产 Web 服务器",
        "org_id": "00000000-0000-0000-0000-000000000002",
        "comment": "外部工单 ITSM-2026072201 审批未通过"
    }'
```

#### 方式 B：外部审批通过后直接创建资产授权

适用于"JumpServer 中完全不走工单"的场景。外部审批通过后，对接程序使用管理员凭证直接创建资产授权，用户立即获得访问权限。

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
        "comment": "对应外部工单号 ITSM-2026072201"
    }'
```

授权名称 `name` 建议带上外部工单号，方便到期清理与审计追溯。到期后授权自动失效（`date_expired`），也可由对接程序调用删除授权接口主动回收。

## 完整示例代码

以下 Python 脚本串起完整流程：确认流程配置、代用户提单、轮询状态；并提供外部审批结果回写（审批/驳回既有工单、直接创建授权）的函数。三种身份的 Token 按需配置。

```python
# -*- coding: utf-8 -*-
"""
JumpServer 对接外部工单系统示例
方向一：代用户提交资产申请工单 + 轮询工单状态
方向二：外部审批通过后审批既有工单，或直接创建资产授权
依赖：pip install requests
"""

import json
import time

import requests

API_URL = "https://localhost"
ORG_ID = "00000000-0000-0000-0000-000000000002"

# 三种身份的 Token：申请人（提单）、审批人（审批既有工单）、管理员（直接创建授权）
APPLICANT_TOKEN = "your applicant token"
APPROVER_TOKEN = "your approver token"
ADMIN_TOKEN = "your admin token"

VERIFY_SSL = False   # 自签名证书环境设为 False，生产环境建议配置可信证书并设为 True

ASSET_ID = "b4f205af-4353-49ef-befa-ff9095d52a27"   # 申请的资产 ID
USER_ID = "8f6e2b1a-3c4d-4e5f-9a0b-1c2d3e4f5a6b"    # 直接授权时的用户 ID
EXTERNAL_TICKET_NO = "ITSM-2026072201"               # 外部系统工单号


def build_headers(token):
    """构造带认证信息的请求头"""
    return {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {token}",
        "X-JMS-ORG": ORG_ID,
    }


def request_api(method, path, token, **kwargs):
    """统一请求入口，包含错误处理"""
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
        # 打印服务端返回的错误详情，便于定位参数问题
        print(f"HTTP 错误: {e}, 响应内容: {e.response.text}")
        raise
    except requests.exceptions.RequestException as e:
        print(f"请求失败: {e}")
        raise


def check_apply_asset_flow():
    """步骤 1：确认 apply_asset 审批流程已配置"""
    result = request_api(
        "GET", "/api/v1/tickets/flows/", APPLICANT_TOKEN,
        params={"type": "apply_asset", "offset": 0, "limit": 15},
    )
    flows = result.get("results", result) if isinstance(result, dict) else result
    if not flows:
        raise RuntimeError("未配置 apply_asset 审批流程，请先在 JumpServer 中配置工单流程")
    print(f"审批流程已配置，共 {len(flows)} 条")


def submit_asset_ticket():
    """步骤 2：代用户提交资产申请工单（使用申请人身份）"""
    data = {
        "title": f"{EXTERNAL_TICKET_NO} 申请访问生产 Web 服务器",
        "org_id": ORG_ID,
        "apply_assets": [ASSET_ID],
        "apply_accounts": ["@ALL"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T00:00:00.000Z",
        "apply_date_expired": "2026-08-22T00:00:00.000Z",
        "comment": f"对应外部工单号 {EXTERNAL_TICKET_NO}",
    }
    ticket = request_api(
        "POST", "/api/v1/tickets/apply-asset-tickets/open/",
        APPLICANT_TOKEN, data=json.dumps(data),
    )
    print(f"工单提交成功, id={ticket['id']}, serial_num={ticket.get('serial_num')}")
    return ticket["id"]


def poll_ticket(ticket_id, interval=30, max_rounds=120):
    """步骤 3：轮询工单状态，直到审批完成或超时"""
    for _ in range(max_rounds):
        ticket = request_api(
            "GET", f"/api/v1/tickets/tickets/{ticket_id}/", APPLICANT_TOKEN,
        )
        state = ticket["state"]["value"]      # pending / approved / rejected
        status = ticket["status"]["value"]    # open / closed
        print(f"当前状态: state={state}, status={status}")
        if state != "pending":
            return state
        time.sleep(interval)
    raise TimeoutError("轮询超时，工单仍处于待审批状态")


def approve_ticket(ticket_id):
    """方向二/方式 A：外部审批通过后，审批既有工单（使用审批人身份）"""
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
    print(f"工单 {ticket_id} 已审批通过")


def reject_ticket(ticket_id):
    """方向二/方式 A：外部审批被拒后，驳回既有工单（使用审批人身份）"""
    data = {
        "title": f"{EXTERNAL_TICKET_NO} 申请访问生产 Web 服务器",
        "org_id": ORG_ID,
        "comment": f"外部工单 {EXTERNAL_TICKET_NO} 审批未通过",
    }
    request_api(
        "PUT", f"/api/v1/tickets/apply-asset-tickets/{ticket_id}/reject/",
        APPROVER_TOKEN, data=json.dumps(data),
    )
    print(f"工单 {ticket_id} 已驳回")


def create_asset_permission():
    """方向二/方式 B：外部审批通过后，直接创建资产授权（使用管理员身份）"""
    data = {
        "name": f"{EXTERNAL_TICKET_NO}-perm",
        "users": [USER_ID],
        "assets": [ASSET_ID],
        "accounts": ["@ALL"],
        "actions": ["connect", "upload", "download"],
        "is_active": True,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2026-08-22T00:00:00.000Z",
        "comment": f"对应外部工单号 {EXTERNAL_TICKET_NO}",
    }
    perm = request_api(
        "POST", "/api/v1/perms/asset-permissions/",
        ADMIN_TOKEN, data=json.dumps(data),
    )
    print(f"资产授权创建成功, id={perm['id']}")
    return perm["id"]


def main():
    # ---- 方向一：代用户提单并轮询 ----
    check_apply_asset_flow()
    ticket_id = submit_asset_ticket()
    final_state = poll_ticket(ticket_id, interval=30)
    print(f"工单最终状态: {final_state}")

    # ---- 方向二：外部审批结果回写（按实际对接场景二选一）----
    # 方式 A：外部审批通过 -> 审批既有工单；被拒 -> 驳回
    # approve_ticket(ticket_id)
    # reject_ticket(ticket_id)

    # 方式 B：不走 JumpServer 工单，直接创建授权
    # create_asset_permission()


if __name__ == "__main__":
    main()
```

## 常见问题

**Q1：提交工单接口返回 400，提示与审批流程相关的错误？**

A：当前组织尚未配置 `apply_asset` 类型的工单审批流程。请先在 JumpServer 页面（工单 - 流程设置）或通过 `/api/v1/tickets/flows/` 接口配置流程后再提单。注意流程按组织隔离，`X-JMS-ORG` 要与提单组织一致。

**Q2：工单的申请人显示的是对接程序的服务账号，而不是实际用户？**

A：工单申请人以调用接口的认证身份为准。要让工单体现真实申请用户，外部系统需要使用该用户本人的 Bearer Token 或 API Key（AK/SK）来调用提单接口，而不是统一用一个服务账号代发。可为每个用户在 JumpServer 中创建 API Key 并托管在对接程序中。

**Q3：调用 approve 接口返回 403 没有权限？**

A：审批接口必须由该工单**当前审批步骤的受理人**身份调用，普通服务账号即使是组织管理员也可能不在受理人列表中。请检查流程 `rules` 中各级受理人配置，并确保对接程序使用的是对应受理人的凭证；多级审批时每一级都需要由该级受理人分别调用一次。

**Q4：`apply_actions` / `actions` 传 `all` 报错，取值到底有哪些？**

A：动作枚举值为 `connect`、`upload`、`download`、`copy`、`paste`、`delete`、`share`，不接受 `all` 这样的聚合值，需要全部动作时逐项列出。另外注意时间字段（`apply_date_start`、`date_expired` 等）为 ISO 8601 datetime 格式，`apply_date_expired` 必须晚于 `apply_date_start`，否则会返回 400。
