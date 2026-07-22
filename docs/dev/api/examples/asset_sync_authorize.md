# 实战案例：外部系统申请资产并自动授权

## 场景说明

企业内部的 CMDB 或运维平台（下称“外部系统”）需要定期从 JumpServer 拉取资产与节点清单，展示给业务用户浏览、检索。用户在外部系统中对某台资产发起使用申请，外部系统走完自身审批流后，调用 JumpServer 接口为该用户创建一条**限时资产授权**（通过 `date_start` / `date_expired` 控制生效与失效时间）。授权到期后自动失效，无需人工回收，从而实现“申请 → 审批 → 授权 → 到期自动失效”的完整闭环。

## 前置条件

- 拥有一个具备管理权限的 API 认证凭据：管理员的 Bearer Token，或系统设置中创建的 API Key（AK/SK，配合 HTTP 签名认证使用）。该凭据需具备资产、用户的查看权限以及资产授权的创建权限。
- 明确资产、用户所在的组织，调用接口时通过请求头 `X-JMS-ORG` 指定组织 ID（不传默认为 `Default` 组织）。
- 申请人已存在于 JumpServer 中（本地用户或 LDAP 等方式同步的用户），且外部系统能拿到其用户名（`username`），用于反查用户 ID。
- 被申请的资产已纳管到 JumpServer，且资产上已配置可用的登录账号（否则授权 `accounts` 无实际可连账号）。
- Python 完整示例需安装依赖：`pip install requests httpsig`。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/assets/nodes/` | 获取节点（资产树）清单，供外部系统同步展示 |
| GET | `/api/v1/assets/assets/` | 分页获取资产清单（支持 `search`/`name`/`address` 等过滤），供外部系统同步展示 |
| GET | `/api/v1/assets/nodes/{id}/assets/` | 获取指定节点下的资产列表（外部系统按节点树展示时使用） |
| GET | `/api/v1/users/users/` | 按 `username` 反查申请人在 JumpServer 中的用户 ID |
| POST | `/api/v1/perms/asset-permissions/` | 审批通过后创建限时资产授权（`date_start` / `date_expired`） |
| GET | `/api/v1/perms/users/{user}/assets/` | 查询用户当前已授权的资产，用于授权结果回验 |
| DELETE | `/api/v1/perms/asset-permissions/{id}/` | （可选）审批撤销或提前回收时删除授权 |

> 以上接口的字段级明细，可在 JumpServer 在线 API 文档 `https://<JumpServer地址>/api/docs` 中查看。

## 操作流程

### 第一步：同步节点与资产清单

外部系统定时（如每小时）拉取节点与资产清单并缓存到本地库，供用户浏览和发起申请。先拉取节点列表，构建资产树结构。

```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/?offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

再分页拉取资产列表，`search` 参数可按名称、地址模糊过滤：

```sh
curl -X GET 'https://localhost/api/v1/assets/assets/?offset=0&limit=100&search=web' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

如果外部系统按节点树逐级展示，也可以按节点拉取该节点下的资产：

```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/3728f004-99a2-4fca-9577-84d5ffcf9eff/assets/?offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

记录返回结果中每个资产的 `id`（UUID），后续创建授权时需要使用。

### 第二步：查询申请人的用户 ID

用户在外部系统提交申请后，外部系统用其用户名反查 JumpServer 用户 ID：

```sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

从返回列表中取出该用户的 `id` 字段（UUID）。若列表为空，说明该用户不在当前组织或尚未同步到 JumpServer，应在外部系统中提示申请人。

### 第三步：审批通过后创建限时授权

外部系统审批通过后，调用创建资产授权接口。关键点：

- `users`：申请人用户 ID 列表；`assets`：被申请资产 ID 列表；
- `accounts`：授权账号，可填账号 ID，或特殊值 `@ALL`（所有账号）、`@SPEC`（指定账号）、`@INPUT`（手动账号）、`@USER`（同名账号）；
- `actions`：授权动作，可选值 `connect`、`upload`、`download`、`copy`、`paste`、`delete`、`share`，按最小权限原则只给 `connect` 即可；
- `date_start` / `date_expired`：授权生效与失效时间（ISO 8601 格式），到期后授权自动失效。

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
            "comment":"外部系统工单 TICKET-1001 审批通过后自动创建"
        }'
```

返回 `201` 表示创建成功，记录返回体中的授权 `id`，外部系统应将其与工单关联，便于后续提前回收或审计。建议 `name` 带上工单号等业务标识，保证可追溯且不重名。

### 第四步：回验用户已授权资产

创建成功后，可查询该用户当前被授权的资产列表进行回验，确认目标资产已出现在结果中，再在外部系统中告知用户“授权已生效”：

```sh
curl -X GET 'https://localhost/api/v1/perms/users/cf7a1f14-0c70-4209-8196-c24cb7ec41a4/assets/?offset=0&limit=15' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第五步：到期自动失效与提前回收（可选）

授权在 `date_expired` 到期后自动失效（授权记录的只读字段 `is_expired` 变为 `true`、`is_valid` 变为 `false`），用户不再能通过该授权连接资产，外部系统**无需**在到期时再调用任何接口。若审批被撤销或需要提前回收，删除该条授权即可：

```sh
curl -X DELETE 'https://localhost/api/v1/perms/asset-permissions/b2c45ce6-6b9c-4270-a073-567ff0e0ade8/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

## 完整示例代码

以下 Python 脚本使用 API Key（AK/SK）签名认证，串起“同步清单 → 反查用户 → 创建限时授权 → 回验”完整流程，可直接作为外部系统对接 JumpServer 的参考实现。

```python
# Python 示例：外部系统申请资产并自动授权（完整流程）
# 依赖：pip install requests httpsig

import sys
import json
from datetime import datetime, timedelta

import requests
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"                            # JumpServer 地址
KEY_ID      = "your id"                                      # API Key ID
KEY_SECRET  = "your secret"                                  # API Key Secret
ORG_ID      = "00000000-0000-0000-0000-000000000002"         # 组织 ID

# ---- 业务入参（实际由外部系统的申请工单传入）----
APPLY_USERNAME   = "zhangsan"          # 申请人在 JumpServer 中的用户名
APPLY_ASSET_NAME = "web-server-01"     # 被申请资产的名称
APPLY_TICKET_NO  = "TICKET-1001"       # 外部系统工单号，用于追溯
PERMIT_DAYS      = 7                   # 授权有效期（天），到期自动失效


def build_auth():
    """构造 AK/SK 的 HTTP 签名认证对象"""
    signature_headers = ['(request-target)', 'accept', 'date']
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = signature_headers
    )


def build_headers():
    """每次请求重新生成 Date 头，签名依赖该值"""
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
    """步骤一：同步节点与资产清单（外部系统可定时执行并缓存）"""
    nodes = api_get("/api/v1/assets/nodes/", params = {"offset": 0, "limit": 100})
    assets = api_get("/api/v1/assets/assets/", params = {"offset": 0, "limit": 100})
    print(f"同步完成：节点 {nodes.get('count', 0)} 个，资产 {assets.get('count', 0)} 台")
    return nodes.get("results", []), assets.get("results", [])


def get_user_by_username(username):
    """步骤二：按用户名反查用户 ID"""
    data = api_get("/api/v1/users/users/", params = {"username": username})
    # 未传分页参数时接口直接返回列表，传了 offset/limit 则返回带 results 的分页对象
    users = data if isinstance(data, list) else data.get("results", [])
    for user in users:
        if user.get("username") == username:
            return user
    return None


def find_asset_by_name(assets, name):
    """从已同步的资产清单中按名称精确定位资产"""
    for asset in assets:
        if asset.get("name") == name:
            return asset
    return None


def create_time_limited_permission(user_id, asset_id, days):
    """步骤三：创建限时资产授权，到期自动失效"""
    now = datetime.utcnow()
    time_form = "%Y-%m-%dT%H:%M:%S.000Z"
    data = {
        # name 带上工单号与日期，保证唯一且可追溯
        "name": f"apply-{APPLY_TICKET_NO}-{now.strftime('%Y%m%d%H%M%S')}",
        "users": [user_id],
        "assets": [asset_id],
        "accounts": ["@ALL"],          # 也可为账号 id，或 @SPEC/@INPUT/@USER
        "actions": ["connect"],        # 最小权限：仅允许连接
        "is_active": True,
        "date_start": now.strftime(time_form),
        "date_expired": (now + timedelta(days = days)).strftime(time_form),
        "comment": f"外部系统工单 {APPLY_TICKET_NO} 审批通过后自动创建"
    }
    return api_post("/api/v1/perms/asset-permissions/", data)


def verify_user_perm_assets(user_id, asset_id):
    """步骤四：回验该用户已授权资产中是否包含目标资产"""
    data = api_get(
        f"/api/v1/perms/users/{user_id}/assets/",
        params = {"offset": 0, "limit": 100}
    )
    permed = data.get("results", [])
    return any(asset.get("id") == asset_id for asset in permed)


def main():
    try:
        # 1. 同步节点与资产清单
        _nodes, assets = sync_nodes_and_assets()

        # 2. 反查申请人用户 ID
        user = get_user_by_username(APPLY_USERNAME)
        if not user:
            print(f"错误：用户 {APPLY_USERNAME} 不存在于当前组织，请确认用户已同步")
            sys.exit(1)
        print(f"申请人：{user['name']}（id={user['id']}）")

        # 3. 定位被申请资产
        asset = find_asset_by_name(assets, APPLY_ASSET_NAME)
        if not asset:
            print(f"错误：资产 {APPLY_ASSET_NAME} 未找到，请确认资产已纳管且组织正确")
            sys.exit(1)
        print(f"目标资产：{asset['name']}（id={asset['id']}）")

        # 4. 创建限时授权（外部系统应在自身审批通过后再执行本步骤）
        perm = create_time_limited_permission(user["id"], asset["id"], PERMIT_DAYS)
        print("限时授权创建成功:")
        print(json.dumps(
            {k: perm.get(k) for k in
             ("id", "name", "date_start", "date_expired", "is_active")},
            indent = 2, ensure_ascii = False
        ))

        # 5. 回验授权结果
        if verify_user_perm_assets(user["id"], asset["id"]):
            print(f"回验通过：{APPLY_USERNAME} 已可访问 {APPLY_ASSET_NAME}，"
                  f"{PERMIT_DAYS} 天后授权自动失效")
        else:
            print("回验未通过：授权已创建但用户暂未查询到该资产，"
                  "请检查组织、date_start 是否为未来时间等")

    except requests.exceptions.HTTPError as e:
        print(f"API 请求失败: {e}")
        if e.response is not None:
            print(f"响应内容: {e.response.text}")
        sys.exit(1)
    except requests.exceptions.RequestException as e:
        print(f"网络错误: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

## 常见问题

**Q1：授权到期后，外部系统还需要调用接口取消授权吗？**

不需要。授权在 `date_expired` 之后自动失效（记录的只读字段 `is_expired` 为 `true`、`is_valid` 为 `false`），用户无法再通过该授权连接资产。到期的授权记录会保留，便于审计；如需清理或提前回收，调用 `DELETE /api/v1/perms/asset-permissions/{id}/` 删除即可。

**Q2：创建授权返回 400，一般是什么原因？**

常见原因：`name` 与已有授权重名（建议带工单号保证唯一）；`date_start` / `date_expired` 时间格式不正确（应使用 ISO 8601 格式，如 `2026-07-22T09:00:00.000Z`）或失效时间早于开始时间；`users` / `assets` 中的 ID 不存在于当前 `X-JMS-ORG` 指定的组织。可根据响应体中的字段错误提示逐项排查。

**Q3：授权创建成功（201），但回验接口查不到该资产？**

依次检查：请求头 `X-JMS-ORG` 是否与创建授权时一致（跨组织查询结果为空）；`date_start` 是否设置成了未来时间（授权尚未生效）；`is_active` 是否为 `true`。另外回验接口路径参数是用户 ID（UUID），传成用户名会导致 404。

**Q4：`accounts` 和 `actions` 应该怎么填？**

`accounts` 可填资产上的账号 ID，也可使用特殊值：`@ALL`（所有账号）、`@SPEC`（指定账号）、`@INPUT`（手动账号）、`@USER`（同名账号）。`actions` 可选值为 `connect`、`upload`、`download`、`copy`、`paste`、`delete`、`share`；对外部系统申请类场景，建议按最小权限原则只授予 `connect`，有文件传输需求再增加 `upload` / `download`。
