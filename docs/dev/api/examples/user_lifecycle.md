# 实战案例：用户生命周期自动化（对接 HR/OA）

## 场景说明

企业通常在 HR/OA 系统中维护员工的入职与离职状态，堡垒机账号若靠人工开通与回收，容易出现「入职开通慢、离职忘回收」的问题，离职人员残留的访问权限更是重大安全隐患。本案例演示如何通过 JumpServer API 与 HR 系统联动：员工入职时自动创建堡垒机用户、按部门加入用户组并继承部门授权；员工离职时自动禁用账号、移出用户组并回收个人直接授权，经过一段观察期确认无遗留问题后再删除账号。HR 系统只需在入职/离职流程节点通过 Webhook 或定时任务触发本文脚本即可。

## 前置条件

- 一个具备管理权限的 API 凭据：组织管理员（或系统管理员）用户的 Bearer Token，或该用户的 API Key（AK/SK，配合 `httpsig` 签名认证），二选一。
- 已知目标组织的组织 ID（请求头 `X-JMS-ORG`，缺省为 `Default` 组织 `00000000-0000-0000-0000-000000000002`）。
- 已在 JumpServer 中规划好与部门对应的用户组（如「运维组」），以及部门需要访问的资产/节点。
- 推荐采用「授权绑定用户组」的模式：授权规则的对象是部门用户组而不是个人。这样入职进组即得到授权、离职出组即回收授权，避免授权碎片化。
- 运行环境：Python 3，安装 `requests`（若使用 AK/SK 认证再安装 `httpsig`）。

所有请求统一携带以下请求头：

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/users/groups/` | 按名称查询用户组，获取部门组 ID |
| POST | `/api/v1/users/users/` | 创建用户（入职开通账号，可同时指定用户组） |
| GET | `/api/v1/users/groups/{id}/` | 查询用户组详情（取现有成员，供合并） |
| PATCH | `/api/v1/users/groups/{id}/` | 更新用户组 `users` 字段（把已有用户加入用户组） |
| POST | `/api/v1/perms/asset-permissions/` | 创建资产授权（按部门用户组授权，首次执行） |
| GET | `/api/v1/users/users/` | 按 `username` 查询用户，定位离职人员账号 |
| PATCH | `/api/v1/users/users/{id}/` | 禁用用户（`is_active=false`）并清空用户组 |
| GET | `/api/v1/perms/asset-permissions/` | 按 `user_id` 查询用户名下的直接授权 |
| GET / PATCH | `/api/v1/perms/asset-permissions/{id}/` | 查询授权详情并从 `users` 中移除该用户 |
| DELETE | `/api/v1/users/users/{id}/` | 删除用户（离职观察期结束后执行） |

> 各接口的字段级明细可在你环境的在线文档 `https://<JumpServer地址>/api/docs` 中查看。

## 操作流程

### 入职流程

#### 步骤 1：查询部门用户组，获取组 ID

按 HR 侧的部门名称查询对应的用户组（例如「运维组」），拿到组 ID 供后续创建用户时使用。

```sh
curl -X GET 'https://localhost/api/v1/users/groups/?name=%E8%BF%90%E7%BB%B4%E7%BB%84' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回列表中取 `id` 字段，例如 `7413d36d-cf37-45f4-b42a-cd5eeff1f4ff`。

#### 步骤 2：创建用户并直接加入用户组

创建用户时通过 `groups` 字段直接指定所属用户组，一步完成「开号 + 入组」。`password_strategy` 建议使用 `email`，由系统向员工邮箱发送密码设置链接，避免脚本中出现明文密码；`comment` 中可回写 HR 工号便于对账。

```sh
curl -X POST 'https://localhost/api/v1/users/users/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "张三",
        "username": "zhangsan",
        "email": "zhangsan@example.com",
        "password_strategy": "email",
        "source": "local",
        "groups": ["7413d36d-cf37-45f4-b42a-cd5eeff1f4ff"],
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}],
        "date_expired": "2096-12-31T00:00:00.000Z",
        "comment": "HR 工号 E10086，入职流程自动创建"
    }'
```

创建成功返回完整用户对象，记录其中的 `id`（用户 UUID）。

#### 步骤 3（可选）：把已存在的用户加入用户组

若账号已存在（如转岗场景），改用 PATCH 更新用户组的 `users` 字段。注意 `users` 是全量替换，必须先 GET 组详情取出现有成员 ID，把新用户 ID 合并进去后再提交，否则会把其他成员移出该组。

```sh
# 先查询组详情，得到现有成员
curl -X GET 'https://localhost/api/v1/users/groups/7413d36d-cf37-45f4-b42a-cd5eeff1f4ff/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# 再用「现有成员 + 新成员」全量提交
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

#### 步骤 4：为部门用户组创建资产授权（每个部门仅需首次执行）

授权对象指向用户组（`user_groups`），资产范围用节点（`nodes`）或资产（`assets`）指定。该授权创建一次即可，之后新员工入组自动继承，无需为每个人重复创建。

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
        "comment": "运维组 Linux 服务器授权，入职自动化流程创建"
    }'
```

`accounts` 中 `@SPEC` 表示指定账号（配合具体账号名使用），也支持 `@ALL` 等特殊值；`actions` 可选值为 `connect`、`upload`、`download`、`copy`、`paste`、`delete`、`share`。

### 离职流程

#### 步骤 5：按用户名定位离职人员账号

HR 系统给出的通常是工号或用户名，先按 `username` 精确查询拿到用户 UUID。

```sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

#### 步骤 6：禁用账号并移出所有用户组（观察期开始）

禁用（`is_active=false`）后该用户立即无法登录；同时把 `groups` 置空，用户随即失去所有来自用户组的授权。不要直接删除——保留账号进入观察期（如 30 天），便于审计追溯与误操作回滚。

```sh
curl -X PATCH 'https://localhost/api/v1/users/users/3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ "is_active": false, "groups": [] }'
```

#### 步骤 7：回收个人名下的直接授权

除组授权外，用户可能还有直接绑定到个人的授权（如工单审批产生的授权）。先按 `user_id` 列出相关授权，再逐条查询详情、从 `users` 中剔除该用户后 PATCH 回去（同样是全量替换）。

```sh
# 列出该用户关联的授权
curl -X GET 'https://localhost/api/v1/perms/asset-permissions/?user_id=3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# 对每条授权：查详情后，把去掉该用户的 users 列表全量提交
curl -X PATCH 'https://localhost/api/v1/perms/asset-permissions/9c8d7e6f-0000-0000-0000-0000000000aa/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ "users": ["5e2d8c0a-1b7f-4a2e-8d3c-9f1a2b3c0002"] }'
```

> 注：按 `user_id` 过滤的结果包含经由用户组关联到的授权。脚本中只需处理 `users` 里确实含有该用户 ID 的授权；组授权在步骤 6 移出用户组时已自动回收。

#### 步骤 8：观察期结束后删除用户

确认观察期内无异常（无审计追溯需求、无误操作申诉）后，删除账号完成闭环。删除成功返回 `204 No Content`。

```sh
curl -X DELETE 'https://localhost/api/v1/users/users/3f9b6bea-63a9-4b3e-9c9a-3c9e0e6f0001/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

## 完整示例代码

```python
# -*- coding: utf-8 -*-
"""
JumpServer 用户生命周期自动化脚本（对接 HR/OA）

入职 onboard : 查询部门用户组 -> 创建用户并入组 -> （首次）为用户组创建资产授权
离职 offboard: 定位用户 -> 禁用并移出所有用户组 -> 回收个人直接授权
删除 purge   : 离职观察期结束后删除用户

认证方式：Bearer Token（如需 AK/SK，可改用 httpsig 的 HTTPSignatureAuth，
参考《用户管理》章节的 Python 示例）
"""

import sys
import requests

API_URL = "https://localhost"                                # JumpServer 地址
TOKEN   = "b96810faac725563304dada8c323c4fa061863d4"         # 管理员 Token
ORG_ID  = "00000000-0000-0000-0000-000000000002"             # 组织 ID

session = requests.Session()
session.headers.update({
    "Authorization": f"Bearer {TOKEN}",
    "X-JMS-ORG": ORG_ID,
    "Content-Type": "application/json",
})
session.verify = False   # 自签名证书环境；生产环境建议配置可信证书后改为 True


def check(resp, action):
    """统一错误处理：非 2xx 直接抛出异常，204 无响应体"""
    if not resp.ok:
        raise RuntimeError(f"{action} 失败: HTTP {resp.status_code} {resp.text}")
    return resp.json() if resp.status_code != 204 else None


def as_results(data):
    """兼容分页（{'results': [...]}）与非分页（[...]）两种返回结构"""
    return data.get("results", []) if isinstance(data, dict) else data


# ---------- 入职 ----------

def get_group_id(group_name):
    """按名称查询用户组，返回组 ID"""
    resp = session.get(f"{API_URL}/api/v1/users/groups/",
                       params={"name": group_name})
    groups = as_results(check(resp, f"查询用户组 {group_name}"))
    if not groups:
        raise RuntimeError(f"用户组不存在: {group_name}，请先在控制台创建")
    return groups[0]["id"]


def create_user(name, username, email, group_id, emp_no):
    """创建用户并直接加入部门用户组，密码通过邮件链接由员工自行设置"""
    payload = {
        "name": name,
        "username": username,
        "email": email,
        "password_strategy": "email",
        "source": "local",
        "groups": [group_id],
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}],
        "comment": f"HR 工号 {emp_no}，入职流程自动创建",
    }
    resp = session.post(f"{API_URL}/api/v1/users/users/", json=payload)
    user = check(resp, f"创建用户 {username}")
    print(f"[入职] 用户已创建并加入用户组: {username} ({user['id']})")
    return user


def ensure_group_permission(group_id, perm_name, node_ids, accounts):
    """确保部门用户组已有资产授权；不存在则创建（每个部门只需一次）"""
    resp = session.get(f"{API_URL}/api/v1/perms/asset-permissions/",
                       params={"name": perm_name})
    if as_results(check(resp, "查询授权")):
        print(f"[入职] 授权已存在，跳过创建: {perm_name}")
        return
    payload = {
        "name": perm_name,
        "user_groups": [group_id],
        "nodes": node_ids,
        "assets": [],
        "accounts": accounts,                      # 如 ["@SPEC", "root"]
        "protocols": ["all"],
        "actions": ["connect", "upload", "download"],
        "is_active": True,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2096-12-31T00:00:00.000Z",
        "comment": "入职自动化流程创建的部门授权",
    }
    resp = session.post(f"{API_URL}/api/v1/perms/asset-permissions/",
                        json=payload)
    perm = check(resp, f"创建授权 {perm_name}")
    print(f"[入职] 部门授权已创建: {perm_name} ({perm['id']})")


def onboard(name, username, email, dept_group, emp_no,
            node_ids, accounts):
    """入职入口：HR 系统在入职节点调用"""
    group_id = get_group_id(dept_group)
    user = create_user(name, username, email, group_id, emp_no)
    ensure_group_permission(group_id, f"{dept_group}_auto_perm",
                            node_ids, accounts)
    return user


# ---------- 离职 ----------

def find_user_by_username(username):
    """按用户名精确定位用户，返回用户对象"""
    resp = session.get(f"{API_URL}/api/v1/users/users/",
                       params={"username": username})
    users = as_results(check(resp, f"查询用户 {username}"))
    matched = [u for u in users if u["username"] == username]
    if not matched:
        raise RuntimeError(f"未找到用户: {username}")
    return matched[0]


def disable_user(user_id):
    """禁用账号并移出所有用户组：立即失去登录能力与全部组授权"""
    resp = session.patch(f"{API_URL}/api/v1/users/users/{user_id}/",
                         json={"is_active": False, "groups": []})
    check(resp, "禁用用户")
    print(f"[离职] 用户已禁用并移出所有用户组: {user_id}")


def revoke_direct_permissions(user_id):
    """回收直接绑定到个人的授权（users 字段全量替换，剔除该用户后回写）"""
    resp = session.get(f"{API_URL}/api/v1/perms/asset-permissions/",
                       params={"user_id": user_id})
    perms = as_results(check(resp, "查询用户关联授权"))
    for perm in perms:
        detail_url = f"{API_URL}/api/v1/perms/asset-permissions/{perm['id']}/"
        detail = check(session.get(detail_url), "查询授权详情")
        old_users = detail.get("users") or []
        remain = [u["id"] for u in old_users if u["id"] != user_id]
        if len(remain) == len(old_users):
            continue   # 该授权经由用户组关联，users 中并无此人，跳过
        resp = session.patch(detail_url, json={"users": remain})
        check(resp, f"回收授权 {detail['name']}")
        print(f"[离职] 已从授权中移除用户: {detail['name']}")


def offboard(username):
    """离职入口：HR 系统在离职节点调用。先禁用观察，不立即删除"""
    user = find_user_by_username(username)
    disable_user(user["id"])
    revoke_direct_permissions(user["id"])
    print(f"[离职] {username} 处理完成，进入观察期，到期后执行 purge 删除")
    return user["id"]


def purge(username):
    """观察期结束后删除用户，操作不可恢复"""
    user = find_user_by_username(username)
    if user.get("is_active"):
        raise RuntimeError(f"用户 {username} 仍处于启用状态，"
                           f"请先执行离职禁用流程，确认无误后再删除")
    resp = session.delete(f"{API_URL}/api/v1/users/users/{user['id']}/")
    check(resp, f"删除用户 {username}")
    print(f"[删除] 用户已删除: {username}")


if __name__ == "__main__":
    # 用法:
    #   python user_lifecycle.py onboard  张三 zhangsan zhangsan@example.com 运维组 E10086
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
            print("未知操作，支持: onboard / offboard / purge")
    except (IndexError, ValueError):
        print("参数不足，请参考脚本头部注释中的用法")
    except RuntimeError as e:
        print(f"执行失败: {e}")
        sys.exit(1)
```

## 常见问题

**Q1：把用户加入用户组后，其他成员突然「消失」了？**
A：`PATCH /api/v1/users/groups/{id}/` 的 `users` 字段和 `PATCH /api/v1/users/users/{id}/` 的 `groups` 字段都是全量替换而非追加。加成员前必须先 GET 现有列表、合并后再提交；本文脚本在创建用户时直接用 `groups` 字段入组，天然避开了这个坑。

**Q2：为什么推荐授权绑定用户组，而不是直接授权给个人？**
A：授权绑定用户组后，入职「进组即生效」、离职「出组即回收」，授权规则数量恒定、易审计；若逐人授权，员工流动会造成授权碎片化，离职时容易漏收。个人直接授权应仅保留给工单等临时场景，并在离职流程中统一回收（见步骤 7）。

**Q3：调用接口返回 403 或查不到刚创建的资源？**
A：先检查 Token 对应用户是否具备组织管理员/系统管理员角色；再检查 `X-JMS-ORG` 是否与目标资源所属组织一致——用户组、授权都是组织级资源，组织 ID 不匹配时表现为 404 或空列表。创建用户时 `system_roles`、`org_roles` 需按环境实际的角色 ID 填写。

**Q4：离职为什么不直接 DELETE，而要先禁用观察期？**
A：删除不可恢复，且会切断与该用户相关的授权关联关系，不利于审计追溯与误操作（如 HR 状态同步出错）回滚。推荐流程：先 `is_active=false` 禁用并回收授权（用户立即无法登录，风险已消除），保留 30 天左右观察期，确认无异常后再执行 DELETE。

**Q5：禁用后，该用户已经打开的在线会话会自动断开吗？**
A：禁用阻止的是后续登录。稳妥起见，离职处理后应在控制台「会话管理」中检查该用户是否存在在线会话并手动终断。
