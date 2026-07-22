## /api/v1/audits/login-logs/

### GET

- **描述：**
查询用户登录日志，即用户通过 Web、终端等方式登录 JumpServer 的审计记录，包含登录来源 IP、城市、认证方式、MFA 状态与成败结果。

> 注：该接口不支持 `date_from` / `date_to` 时间范围参数，时间过滤需在客户端根据返回的 `datetime` 字段自行处理；可配合 `order=-datetime` 按登录时间倒序分页拉取后再截取所需时间段。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| username | 类型：String，按用户名过滤 | - |
| ip | 类型：String，按登录来源 IP 过滤 | - |
| city | 类型：String，按登录城市过滤 | - |
| type | 类型：String，登录来源类型 | W（Web）/ T（Terminal）/ U（Unknown） |
| status | 类型：int，登录状态 | 1（成功）/ 0（失败） |
| mfa | 类型：int，MFA 状态 | 0（禁用）/ 1（启用）/ 2 |
| id | 类型：String(UUID)，按日志 ID 过滤 | - |
| search | 类型：String，搜索词（可填用户名、IP 等模糊匹配） | - |
| order | 类型：String，排序字段 | 如 `datetime`（升序）/ `-datetime`（降序） |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，登录日志列表 | 列表元素为登录日志对象(见下) |
| id | 类型：string，日志ID | UUID |
| username | 类型：string，用户名 | 格式如 `名称(用户名)` |
| type | 类型：object，登录来源类型 | {"value":"W","label":"Web"} 等 |
| ip | 类型：string，登录来源IP |  |
| city | 类型：string，登录城市 | 内网地址显示为 `局域网` |
| user_agent | 类型：string，用户代理 | 浏览器/客户端 UA 信息 |
| mfa | 类型：object，MFA 等级 | {"value":0,"label":"禁用"} 等 |
| reason | 类型：string，失败原因 | 登录成功时为空字符串 |
| reason_display | 类型：string，失败原因说明 |  |
| backend | 类型：string，认证后端 | 如 Password |
| backend_display | 类型：string，认证后端说明 | 如 密码 |
| status | 类型：object，登录状态 | {"value":true,"label":"成功"} 等 |
| datetime | 类型：string(date-time)，登录时间 | 客户端时间过滤依据此字段 |

- **响应示例：**

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "19bf02fa-4cfd-4c03-8162-e62c20cf9505",
            "username": "skj(skj)",
            "type": {
                "value": "W",
                "label": "Web"
            },
            "ip": "10.1.240.254",
            "city": "局域网",
            "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
            "mfa": {
                "value": 0,
                "label": "禁用"
            },
            "reason": "",
            "reason_display": "",
            "backend": "Password",
            "backend_display": "密码",
            "status": {
                "value": true,
                "label": "成功"
            },
            "datetime": "2024/01/22 10:17:20 +0800"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?offset=0&limit=15&order=-datetime' \
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

def get_login_logs():
    url = f"{API_URL}/api/v1/audits/login-logs/"
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
        "offset": 0,
        "limit": 15,
        "order": "-datetime"
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers, params = params
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API 请求失败:{e}")
        return None

if __name__ == "__main__":
    result = get_login_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：安全组接到告警排查可疑登录：拉取用户 zhangsan 的失败登录记录并按登录时间倒序分页，在客户端根据 `datetime` 字段截取最近 24 小时的数据，结合来源 IP 与认证后端判断是否存在暴力破解。

```sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?username=zhangsan&status=0&order=-datetime&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：审计日志对接 SIEM 平台](../examples/siem_export.md)
