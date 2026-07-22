## /api/v1/audits/password-change-logs/

### GET

- **描述：**
查询改密日志，即审计台“改密日志”页对应接口，记录 JumpServer 用户密码的变更历史，包含被改密的用户、改密人（更改者）、来源 IP 与改密时间。资产账号的密码轮换记录属于「账号改密」功能，参考[改密计划接口](../pam/changepwd.md)，不在本接口范围内。

> 注：该接口不支持 `date_from` / `date_to` 时间范围参数，时间过滤需在客户端根据返回的 `datetime` 字段自行处理；可配合 `order=-datetime` 按改密时间倒序分页拉取后再截取所需时间段。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| user | 类型：String，按被改密的用户过滤 | - |
| change_by | 类型：String，按改密人（更改者）过滤 | - |
| remote_addr | 类型：String，按来源 IP 过滤 | - |
| search | 类型：String，搜索词（可填用户、改密人、IP 等模糊匹配） | - |
| order | 类型：String，排序字段 | 如 `datetime`（升序）/ `-datetime`（降序） |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，改密日志列表 | 列表元素为改密日志对象(见下) |
| id | 类型：string，日志ID | UUID |
| user | 类型：string，被改密的用户 | 格式如 `名称(用户名)` |
| change_by | 类型：string，改密人（更改者） | 执行改密操作的用户 |
| remote_addr | 类型：string，来源IP | 可为 null |
| datetime | 类型：string(date-time)，改密时间 | 客户端时间过滤依据此字段 |

- **响应示例：**

> 示例为示意数据，字段以在线 /api/docs 为准。

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "3f7b9c2e-8d41-4a5b-9c6d-1e2f3a4b5c6d",
            "user": "张三(zhangsan)",
            "change_by": "Administrator(admin)",
            "remote_addr": "10.1.240.254",
            "datetime": "2026/07/01 10:17:20 +0800"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/password-change-logs/?offset=0&limit=15&order=-datetime' \
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

def get_password_change_logs():
    url = f"{API_URL}/api/v1/audits/password-change-logs/"
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
    result = get_password_change_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：等保检查前，安全组需要导出本季度的用户改密记录作为合规凭证：可用 `search` 按用户名缩小范围（如核查管理员 admin），按改密时间倒序分页拉取，在客户端根据 `datetime` 字段截取本季度数据后归档留存。

```sh
curl -X GET 'https://localhost/api/v1/audits/password-change-logs/?search=admin&order=-datetime&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：审计日志对接 SIEM 平台](../examples/siem_export.md)
