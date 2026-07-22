## /api/v1/users/groups/

### POST

- **描述：**
创建用户组

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **请求体参数（Body）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| name* | 类型：string，名称 | - |
| users | 类型：object[]，用户；每个元素包含 id（string）、name（string）、is_service_account（boolean） | - |

> 注：带 * 的参数为必填项。
- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，用户组id |  |
| name | 类型：String，用户组名称 |  |
| comment | 类型：String，备注 |  |
| created_by | 类型：String，创建者 |  |
| users | 类型：Object[]，用户；每个元素包含 id（string）、name（string）、is_service_account（boolean） |  |
| org_id | 类型：String，组织 |  |
| org_name | 类型：String，组织名称 |  |
| date_created | 类型：String，创建时间 |  |

- **请求示例**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/users/groups/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{"name":"api_test"}'
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

def create_users_group():
    url = f"{API_URL}/api/v1/users/groups/"
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
        "name": "api_test"
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data),
            verify = False
        )
        response.raise_for_status()
        print("用户组创建成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    create_users_group()
```

- **使用案例：**

场景：公司新组建数据库运维团队，管理员创建 `dba-team` 用户组，并在创建时直接将成员 zhangsan、lisi 加入该组，便于后续按组批量授权数据库资产。

```sh
curl -X POST 'https://localhost/api/v1/users/groups/' \
    -H 'Authorization: Bearer <token>' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "name": "dba-team",
        "users": [
            {"id": "0f6a3fea-1b8f-4e4a-9b8e-2f4d5c6a7b8c", "name": "zhangsan"},
            {"id": "9c2d4e6f-3a5b-4c7d-8e9f-1a2b3c4d5e6f", "name": "lisi"}
        ]
    }'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)

## /api/v1/users/groups/{id}/

### DELETE

- **描述：**
删除用户组

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **请求示例**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/users/groups/7413d36d-cf37-45f4-b42a-cd5eeff1f4ff/' \
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
GROUP_ID    = "your group id"

def delete_users_group():
    url = f"{API_URL}/api/v1/users/groups/{GROUP_ID}/"
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
            url, auth = auth, headers = headers,
            verify = False
        )
        response.raise_for_status()
        print("用户组删除成功")
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    delete_users_group()
```

- **使用案例：**

场景：外包驻场项目验收结束，管理员删除临时用户组 `outsource-temp`（组 ID 已提前查询获得），及时回收该组关联的资产授权，避免权限残留。

```sh
curl -X DELETE 'https://localhost/api/v1/users/groups/5e8b2c1d-4f6a-4b3c-9d2e-7a8b9c0d1e2f/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)
