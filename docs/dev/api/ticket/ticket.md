## /api/v1/tickets/apply-asset-tickets/open/

### POST

- **描述：**
创建工单

- **请求头（Headers）：**  

| 键              | 值                           | 备注                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的token 信息。       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 组织 ID，留空则默认为 Default 组织 |
| Content-Type    | application/json                        | 输出为json格式              

- **请求体参数（Body）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| title* | 类型：String，工单标题 | - |
| org_id* | 类型：String，组织 | - |
| apply_nodes | 类型：Object[]（对象数组，每个元素含 id、name 字段），申请节点 | 支持模糊搜索，最多显示10项 |
| apply_assets | 类型：string[]，申请资产id | 支持模糊搜索，最多显示10项 |
| apply_accounts | 类型：String[]，申请账号id | "@ALL"：所有账号；"@SPEC"：指定账号；"@INPUT"：手动账号；"@USER"：同名账号 |
| apply_actions | 类型：String[]，动作 | 默认：[]；可选值：[connect, upload, download, copy, paste, delete, share] |
| apply_date_start | 类型：String(datetime)，开始日期 | - |
| apply_date_expired | 类型：String(datetime)，失效日期（原文“失效日志”应为笔误） | - |
| comment | 类型：String，备注 | - |
> 注：带 * 的参数为必填项。


- **返回参数:**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，id |  |
| title | 类型：String，标题 |  |
| org_id | 类型：String，组织 |  |
| comment | 类型：String，备注 |  |
| type | 类型：String，类型 |  |
| apply_nodes | 类型：String[]，申请节点 |  |
| apply_assets | 类型：String[]，申请资产 |  |
| apply_accounts | 类型：String[]，申请账号 |  |
| apply_actions | 类型：String[]，申请动作 |  |
| serial_num | 类型：String，序列号 |  |
| approval_step | 类型：String，流程步骤 |  |
| state | 类型：String，工单动作 |  |
| status | 类型：String，工单状态 |  |
| applicant | 类型：String，申请人 |  |
| org_name | 类型：String，组织名称 |  |
| apply_permission_name | 类型：String，工单授权名称 |  |
| apply_date_start | 类型：String(date-time)，申请开始时间 |  |
| apply_date_expired | 类型：String(date-time)，申请结束时间 |  |


- **请求示例：**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/tickets/apply-asset-tickets/open/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "title":"test_tickets_1",
        "apply_accounts":["@ALL"],
        "apply_actions":["connect"],
        "org_id":"00000000-0000-0000-0000-000000000002",
        "apply_assets":["b4f205af-4353-49ef-befa-ff9095d52a27"],
        "apply_date_start":"2023-03-28T02:10:23.245Z",
        "apply_date_expired":"2023-04-04T02:10:23.245Z"
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

def apply_asset_tickets():
    url = f"{API_URL}/api/v1/tickets/apply-asset-tickets/open/"
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
        "title":"test_tickets_1",
        "apply_accounts":["@ALL"],
        "apply_actions":["connect"],
        "org_id": ORG_ID,
        "apply_date_start":"2025-01-01T00:00:00.245Z",
        "apply_date_expired":"2095-01-01T00:00:00.245Z"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("工单创建成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    apply_asset_tickets()
```

- **使用案例：**

场景：外部 ITSM 系统审批立项后，自动为数据库管理员 zhangsan 在 JumpServer 中发起资产访问申请，申请以同名账号连接生产数据库 mysql-prod-01，授权有效期一周。

```sh
curl -X POST 'https://localhost/api/v1/tickets/apply-asset-tickets/open/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "title": "zhangsan 申请访问生产数据库 mysql-prod-01",
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["c3d5f1a2-7b8e-4f6d-9a0c-1e2f3a4b5c6d"],
        "apply_accounts": ["@USER"],
        "apply_actions": ["connect"],
        "apply_date_start": "2026-07-22T09:00:00.000Z",
        "apply_date_expired": "2026-07-29T09:00:00.000Z",
        "comment": "ITSM 工单 INC-20260722-001 关联申请"
    }'
```

> 完整集成场景可参考：[实战案例：对接外部工单系统](../examples/external_ticket.md)

## /api/v1/tickets/tickets/

### GET
- **描述：**
获取工单

- **请求头（Headers）：**  

| 键              | 值                                      | 备注                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的token 信息。       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 组织 ID，留空则默认为 Default 组织。 |
| Content-Type    | application/json                        | 输出为json格式                                                       |

- **查询参数（Query）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| state | 类型：String，动作 | - |
|  | 可选值：pending（待处理）、approved（已同意）、rejected（已拒绝） |  |
| status | 类型：String，状态 | - |
|  | 可选值：closed（已关闭、拒绝）、open（打开） |  |
| type | 类型：String，类型 | - |
|  | 可选值：apply_asset（申请资产）、login_confirm（用户登录复核）、command_confirm（命令复核）、login_asset_confirm（资产登录复核） |  |

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，id |  |
| title | 类型：String，标题 |  |
| org_id | 类型：String，组织 |  |
| serial_num | 类型：String，编号 |  |
| approval_step | 类型：String，工单审批步骤 |  |
| type | 类型：String，类型 |  |
| state | 类型：String，动作 |  |
| applicant | 类型：String，申请人 |  |
| status | 类型：String，状态 |  |
| org_name | 类型：String，组织名称 |  |
| date_created | 类型：String(date-time)，创建时间 |  |
| date_updated | 类型：String(date-time)，更新时间 |  |


- **请求示例**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/?state=pending&status=open&type=apply_asset' \
    -H 'Content-Type: application/json' \
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

def search_tickets():
    url = f"{API_URL}/api/v1/tickets/tickets/"
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

    params = {
        "state": "pending",
        "status": "open",
        "type": "apply_asset"
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        nodes_data = response.json()
        if nodes_data.get("count", 0) == 0:
            print(f"未找到工单")
        else:
            print(f"查询到 {nodes_data['count']} 个匹配的工单：")
            print(json.dumps(nodes_data, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    search_tickets()
```

- **使用案例：**

场景：值班巡检脚本每小时轮询一次所有处于打开状态、待处理的资产登录复核工单，发现积压后通过企业微信提醒值班管理员及时处理。

```sh
curl -X GET 'https://localhost/api/v1/tickets/tickets/?state=pending&status=open&type=login_asset_confirm' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：对接外部工单系统](../examples/external_ticket.md)

## /api/v1/tickets/apply-asset-tickets/{id}/approve/

### PATCH
- **描述：**
审批工单

- **请求头（Headers）：**  

| 键              | 值                                      | 备注                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的token 信息。       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 为组织 ID，此 id号为默认组织：Default，留空则默认为 Default 组织。 |
| Content-Type    | application/json                        | 输出为json格式                                                       |

- **请求体参数（Body）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| org_id | 类型：String，组织id | - |
| apply_nodes | 类型：String[]，申请节点id | - |
| apply_assets | 类型：string[]，申请资产id | - |
| apply_accounts | 类型：String[]，申请账号id | "@ALL"：所有账号；"@SPEC"：指定账号；"@INPUT"：手动账号；"@USER"：同名账号 |
| apply_actions | 类型：String[]，动作 | 默认：[]；可选值：[connect, upload, download, copy, paste, delete, share] |
| apply_date_start | 类型：String(datetime)，开始日期（原文格式“String(date time)”修正为标准datetime格式表述） | - |
| apply_date_expired | 类型：String(datetime)，失效日期（原文“失效日志”应为笔误，格式“String(date time)”修正为标准datetime格式表述） | - |
| comment | 类型：String，备注 | - |

- **请求示例**

**CURL**
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
        "apply_date_start": "2025-03-28 00:00:00",
        "apply_date_expired": "2025-04-04 00:00:00"
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
TICKET_ID   = "your ticket id"
ASSET_ID    = "your asset id"

def approve_tickets():
    url = f"{API_URL}/api/v1/tickets/apply-asset-tickets/{TICKET_ID}/approve/"
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
            "org_id": ORG_ID,
            "apply_assets": [ASSET_ID],
            "apply_accounts": ["@ALL"],
            "apply_actions": ["connect"],
            "apply_date_start": "2025-03-28 00:00:00",
            "apply_date_expired": "2025-04-04 00:00:00"
    }

    try:
        response = requests.patch(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("工单审批成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    approve_tickets()
```

- **使用案例：**

场景：外部工单系统中经理审批通过后，回调 JumpServer 自动放行对应的资产申请工单，仅授予 web-server-01 的连接与文件下载权限，并将授权收紧到本周五下班前失效。

```sh
curl -X PATCH 'https://localhost/api/v1/tickets/apply-asset-tickets/9f8e7d6c-5b4a-3210-fedc-ba9876543210/approve/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "org_id": "00000000-0000-0000-0000-000000000002",
        "apply_assets": ["a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
        "apply_accounts": ["@SPEC"],
        "apply_actions": ["connect", "download"],
        "apply_date_start": "2026-07-22 09:00:00",
        "apply_date_expired": "2026-07-24 18:00:00",
        "comment": "外部工单 INC-20260722-001 审批通过，自动放行"
    }'
```

> 完整集成场景可参考：[实战案例：对接外部工单系统](../examples/external_ticket.md)

## /api/v1/tickets/flows/

### GET
- **描述：**
查询流程

- **请求头（Headers）：**  

| 键              | 值                                      | 备注                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的token 信息。       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 00000000-0000-0000-0000-000000000002 为组织 ID，此 id号为默认组织：Default，留空则默认为 Default 组织。 |
| Content-Type    | application/json                        | 输出为json格式      

- **查询参数（Query）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **请求示例**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/tickets/flows/?offset=0&limit=15' \
    -H 'Content-Type: application/json' \
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

def search_flows():
    url = f"{API_URL}/api/v1/tickets/flows/"
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
        nodes_data = response.json()
        if nodes_data.get("count", 0) == 0:
            print(f"未找到流程")
        else:
            print(f"查询到 {nodes_data['count']} 个匹配的流程：")
            print(json.dumps(nodes_data, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    search_flows()
```

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，id |  |
| type | 类型：Object，流程类型 |  |
| org_id | 类型：String，组织 |  |
| org_name | 类型：String，组织名称 |  |
| approval_level | 类型：int，审批级别 |  |
| rules | 类型：Array，审批流程 | 元素为 TicketFlowApprove 对象，包含 level（类型：int，审批级别，只读）和 users（审批用户）两个字段 |
| created_by | 类型：String，创建人 |  |
| date_created | 类型：String(date-time)，创建时间 |  |
| date_updated | 类型：String(date-time)，更新时间 |  |

- **使用案例：**

场景：对接外部工单系统前的准备阶段，集成脚本分页拉取当前组织已配置的审批流程，核对各类工单的审批级别与审批人是否符合公司安全规范。

```sh
curl -X GET 'https://localhost/api/v1/tickets/flows/?offset=0&limit=10' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：对接外部工单系统](../examples/external_ticket.md)

## /api/v1/tickets/flows/{id}/
### PATCH
- **描述：**
更新流程

- **请求头（Headers）：**  

| 键              | 值                                      | 备注                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------|
| Authorization   | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的token 信息。       |
| X-JMS-ORG       | 00000000-0000-0000-0000-000000000002    | 组织 ID，留空则默认为 Default 组织。 |
| Content-Type    | application/json                        | 输出为json格式                           

- **请求体参数（Body）：**  

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| type | 类型：string，类型 | - |
| approval_level | 类型：int，审批级别 | - |
| rules | 类型：Array，元素为 TicketFlowApprove 对象（含 level：int，审批级别，readOnly 仅响应返回；users：审批人，唯一可写字段） | - |

**请求示例**

**CURL**
```sh
curl -X PATCH 'https://localhost/api/v1/tickets/flows/41b36621-dd4d-492e-a72c-be20b2daeea8/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "type": "apply_asset",
        "approval_level": 1,
        "rules": [{
            "users": []
        }]
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
TICKET_ID   = "your ticket id"

def update_tickets_flows():
    url = f"{API_URL}/api/v1/tickets/flows/{TICKET_ID}/"
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
        "type": "apply_asset",
        "approval_level": 1,
        "rules": [{
            "users": []
        }]
    }

    try:
        response = requests.patch(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("修改流程成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    update_tickets_flows()
```

- **返回参数:**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，id |  |
| type | 类型：Object，流程类型 |  |
| org_id | 类型：String，组织 |  |
| org_name | 类型：String，组织名称 |  |
| approval_level | 类型：int，审批级别 |  |
| rules | 类型：Array，审批流程 |  |
| level | 类型：int，流程级别 |  |
| strategy | 类型：object，审批角色 |  |
| assignees_display | 类型：Array，审批人名称 |  |
| created_by | 类型：String，创建人 |  |
| date_created | 类型：String(date-time)，创建时间 |  |
| date_updated | 类型：String(date-time)，更新时间 |  |

- **使用案例：**

场景：安全整改要求资产申请类工单必须两级审批，运维平台将该流程的审批级别调整为 2 级，并指定安全组审批人 lisi 作为第二级审批人。

```sh
curl -X PATCH 'https://localhost/api/v1/tickets/flows/6d5c4b3a-2f1e-0d9c-8b7a-654321fedcba/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "type": "apply_asset",
        "approval_level": 2,
        "rules": [
            {"users": ["f0e1d2c3-b4a5-6789-0123-456789abcdef"]},
            {"users": ["1a2b3c4d-5e6f-7a8b-9c0d-e1f2a3b4c5d6"]}
        ]
    }'
```

> 完整集成场景可参考：[实战案例：对接外部工单系统](../examples/external_ticket.md)
