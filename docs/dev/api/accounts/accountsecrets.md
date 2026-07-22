## /api/v1/accounts/account-secrets/{id}/

### GET

- **描述：**
查询指定账号的密码/密钥（账号密文）

> 注意：默认情况下查看密码需要 MFA 认证。如需通过 API 直接调用该接口，需要在 `/opt/jumpserver/config/config.txt` 配置文件中增加参数 `SECURITY_VIEW_AUTH_NEED_MFA=False`，并执行 `jmsctl restart` 重启服务后生效。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **路径参数：**

| 参数名 | 类型 | 描述 | 是否必选 |
| --- | --- | --- | --- |
| id | String | 账号 ID，可通过 `/api/v1/accounts/accounts/` 接口查询获取 | 是 |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| id | 类型：string，账号 ID | UUID |
| name | 类型：string，账号名称 |  |
| username | 类型：string，账号用户名 |  |
| secret_type | 类型：object，密文类型 | {"value":"password","label":"密码"} 等 |
| secret | 类型：string，账号密码/密钥明文 | 本接口核心返回字段 |
| asset | 类型：object，所属资产 | 含资产 ID、名称、地址、平台等信息 |
| privileged | 类型：boolean，是否特权账号 |  |
| connectivity | 类型：object，可连接性 | {"value":"ok","label":"成功"} 等 |
| has_secret | 类型：boolean，是否已托管密文 |  |
| is_active | 类型：boolean，是否启用 |  |
| version | 类型：int，密文版本 | 每次改密后版本递增 |
| source | 类型：object，账号来源 | local（数据库）/collected（收集）等 |
| date_created | 类型：string(date-time)，创建时间 |  |
| date_updated | 类型：string(date-time)，更新时间 |  |

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/accounts/account-secrets/1e28b088-c86c-41b7-a85b-29e15c7cc8cb/' \
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
ACCOUNT_ID  = "1e28b088-c86c-41b7-a85b-29e15c7cc8cb"

def get_account_secret(account_id):
    url = f"{API_URL}/api/v1/accounts/account-secrets/{account_id}/"
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
        return response.json()
    except requests.RequestException as e:
        print(f"API 请求失败:{e}")
        return None

if __name__ == "__main__":
    result = get_account_secret(ACCOUNT_ID)
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：数据库巡检脚本每晚连接 db-mysql-01 执行备份前，先按账号 ID 从 JumpServer 实时取回该资产上 root 账号的密码，避免在脚本中硬编码口令（账号 ID 已提前通过 `/api/v1/accounts/accounts/` 按资产名与用户名过滤查得）。

```sh
curl -s -X GET 'https://localhost/api/v1/accounts/account-secrets/f3a9c2d1-7b64-4e0a-9c3f-5d8e2a1b6c40/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' | jq -r '.secret'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)
