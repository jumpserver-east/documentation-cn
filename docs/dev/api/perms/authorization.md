## /api/v1/perms/asset-permissions/

### POST
- **描述：**
创建资产授权

- **请求头（Headers）：**  

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **请求体参数（Body）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| name* | 类型：String，名称 | - |
| users | 类型：String[]，用户id | - |
| user_groups | 类型：String[]，用户组id | - |
| assets | 类型：String[]，资产id | - |
| nodes | 类型：String[]，节点id | - |
| accounts | 类型：String[]，账号id，可选特殊值：<br>- "@ALL"：所有账号<br>- "@SPEC"：指定账号<br>- "@INPUT"：手动账号<br>- "@USER"：同名账号 | - |
| actions | 类型：String[]，动作，可选值：[connect, upload, download, copy, paste, delete, share] | connect, upload, download, copy, paste, delete, share |
| is_active | 类型：Boolean，激活中 | true |
| date_start | 类型：String(datetime)，开始时间 | - |
| date_expired | 类型：String(datetime)，失效时间 | - |
| comment | 类型：String，备注 | - |

> 注：带 * 的参数为必填项。
- **请求示例**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
-H 'Content-Type:application/json' \
-H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
-H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
-d '{
        "name":"create_asset_permission",
        "user_groups":["745980b1-54be-4c5e-b6ab-89826c2e2054"],
        "nodes":["3728f004-99a2-4fca-9577-84d5ffcf9eff"],
        "accounts":["@ALL"],
        "actions":["connect","upload","download","copy","paste"],
        "is_active":true,
        "date_start":"2025-01-01T00:00:00.879Z",
        "date_expired":"2095-01-01T00:00:00.879Z"
    }'
```

**Python**
```python
# Python 示例

import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"
KEY_ID      = "your id"
KEY_SECRET  = "your secret"
ORG_ID      = "your org id"
GROUP_ID    = "your group id"
NODE_ID     = "your node id"

def create_asset_permissions():
    url = f"{API_URL}/api/v1/perms/asset-permissions/"
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
        "name":"create_asset_permission",
        "user_groups":[GROUP_ID],
        "nodes":[NODE_ID],
        "accounts":["@ALL"],
        "actions":["connect","upload","download","copy","paste"],
        "is_active": True,
        "date_start":"2025-01-01T00:00:00.879Z",
        "date_expired":"2095-01-01T00:00:00.879Z"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("资产授权创建成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    create_asset_permissions()
```

- **返回参数：**  

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，id |  |
| name | 类型：String，名称 |  |
| users | 类型：String[]，用户 |  |
| user_groups | 类型：String[]，用户组 |  |
| assets | 类型：String[]，资产 |  |
| nodes | 类型：String[]，节点 |  |
| accounts | 类型：String[]，授权账号 |  |
| actions | 类型：String[]，动作 |  |
| is_active | 类型：Boolean，激活中 |  |
| created_by | 类型：String，创建者 |  |
| comment | 类型：String，备注 |  |
| date_created | 类型：String(date-time)，授权规则创建时间 |  |
| date_start | 类型：String(date-time)，授权规则开始时间 |  |
| date_expired | 类型：String(date-time)，授权规则失效时间 |  |

- **使用案例：**

场景：新员工 zhangsan 入职运维组，管理员为其单独授权 web-server-01、web-server-02 两台资产，仅允许使用与其用户名同名的账号连接（不开放文件传输），授权 90 天后自动失效。

```sh
curl -X POST 'https://localhost/api/v1/perms/asset-permissions/' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer <token>' \
-H 'X-JMS-ORG: <组织ID>' \
-d '{
        "name": "zhangsan-web-servers-90d",
        "users": ["8b1f0a2e-3c4d-4e5f-9a6b-7c8d9e0f1a2b"],
        "assets": ["1a2b3c4d-5e6f-4a7b-8c9d-0e1f2a3b4c5d", "5d4c3b2a-1f0e-4d9c-8b7a-6f5e4d3c2b1a"],
        "accounts": ["@USER"],
        "actions": ["connect"],
        "is_active": true,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2026-10-20T00:00:00.000Z",
        "comment": "新员工 zhangsan 入职授权，仅限同名账号连接，90 天后到期"
    }'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)

## /api/v1/perms/asset-permissions/{id}/

### DELETE
- **描述：**
删除资产授权

- **请求头（Headers）：**  

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **路径参数（Path Params）：**  

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：String，授权ID | 是 |

- **请求示例**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/perms/asset-permissions/b2c45ce6-6b9c-4270-a073-567ff0e0ade8/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' 
```
**Python**
```python
# Python 示例

import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"
KEY_ID      = "your id"
KEY_SECRET  = "your secret"
ORG_ID      = "your org id"
PER_ID      = "your permission id"

def delete_assets_permissions():
    url = f"{API_URL}/api/v1/perms/asset-permissions/{PER_ID}/"
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
        print("资产授权删除成功")
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    delete_assets_permissions()
```

- **使用案例：**

场景：外包人员 wangwu 参与的项目已交付验收，安全审计要求当天回收其临时资产授权（授权 ID 为 f3d2c1b0-a9e8-4d76-b543-2f1e0d9c8b7a）。

```sh
curl -X DELETE 'https://localhost/api/v1/perms/asset-permissions/f3d2c1b0-a9e8-4d76-b543-2f1e0d9c8b7a/' \
-H 'Authorization: Bearer <token>' \
-H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)

### PUT
- **描述：**
更新资产授权

- **请求头（Headers）：**  

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **请求体参数（Body）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| name* | 类型：String，名称 | - |
| users | 类型：String[]，用户id | - |
| user_groups | 类型：String[]，用户组id | - |
| assets | 类型：String[]，资产id | - |
| nodes | 类型：String[]，节点id | - |
| accounts | 类型：String[]，账号，可选特殊值：<br>- "@ALL"：所有账号<br>- "@SPEC"：指定账号<br>- "@INPUT"：手动账号<br>- "@USER"：同名账号 | - |
| actions | 类型：String[]，动作，可选值：[connect, upload, download, copy, paste, delete, share] | connect, upload, download, copy, paste, delete, share |
| is_active | 类型：Boolean，激活中 | true |
| date_start | 类型：String(datetime)，开始时间 | - |
| date_expired | 类型：String(datetime)，失效时间 | - |
| comment | 类型：String，备注 | - |

> 注：带 * 的参数为必填项。
- **请求示例**

**CURL**
```sh
curl -X PUT 'https://localhost/api/v1/perms/asset-permissions/ca90421b-bc45-48c0-bd49-4a213ff61b7c/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "name":"create_asset_permission_update",
            "user_groups":["745980b1-54be-4c5e-b6ab-89826c2e2054"],
            "nodes":["3728f004-99a2-4fca-9577-84d5ffcf9eff"],
            "accounts":["@ALL"],
            "actions":["connect","upload","download","copy","paste"],
            "is_active":true,
            "date_start":"2025-01-01T00:00:00.879Z",
            "date_expired":"2095-01-01T00:00:00.879Z"
        }'
```

**Python**
```python
# Python 示例

import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"
KEY_ID      = "your id"
KEY_SECRET  = "your secret"
ORG_ID      = "your org id"
GROUP_ID    = "your group id"
NODE_ID     = "your node id"
PER_ID      = "your permission id"

def update_asset_permissions():
    url = f"{API_URL}/api/v1/perms/asset-permissions/{PER_ID}/"
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
        "name":"create_asset_permission_update",
        "user_groups":[GROUP_ID],
        "nodes":[NODE_ID],
        "accounts":["@ALL"],
        "actions":["connect","upload","download","copy","paste"],
        "is_active": True,
        "date_start":"2025-01-01T00:00:00.879Z",
        "date_expired":"2095-01-01T00:00:00.879Z"
    }

    try:
        response = requests.put(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("资产授权更新成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    update_asset_permissions()
```

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，id |  |
| name | 类型：String，名称 |  |
| users | 类型：String[]，用户 |  |
| user_groups | 类型：String[]，用户组 |  |
| assets | 类型：String[]，资产 |  |
| nodes | 类型：String[]，节点 |  |
| accounts | 类型：String[]，授权账号 |  |
| actions | 类型：String[]，动作 |  |
| is_active | 类型：Boolean，激活中 |  |
| created_by | 类型：String，创建者 |  |
| comment | 类型：String，备注 |  |
| date_created | 类型：String(date-time)，授权规则创建时间 |  |
| date_start | 类型：String(date-time)，授权规则开始时间 |  |
| date_expired | 类型：String(date-time)，授权规则失效时间 |  |

- **使用案例：**

场景：数据库运维组（dba-team）对生产 MySQL 节点的授权即将到期，管理员将其整体续期一年，并配合本周发布窗口临时增加文件上传、下载动作。

```sh
curl -X PUT 'https://localhost/api/v1/perms/asset-permissions/6e5d4c3b-2a19-4f08-97e6-d5c4b3a29180/' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer <token>' \
-H 'X-JMS-ORG: <组织ID>' \
-d '{
        "name": "dba-team-mysql-prod",
        "user_groups": ["9a8b7c6d-5e4f-4a3b-8c2d-1e0f9a8b7c6d"],
        "nodes": ["2f3e4d5c-6b7a-4c8d-9e0f-1a2b3c4d5e6f"],
        "accounts": ["@ALL"],
        "actions": ["connect", "upload", "download"],
        "is_active": true,
        "date_start": "2026-07-22T00:00:00.000Z",
        "date_expired": "2027-07-22T00:00:00.000Z",
        "comment": "授权续期一年，发布窗口临时开放文件上传下载"
    }'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)
