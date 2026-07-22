## /api/v1/audits/operate-logs/

### GET

- **描述：**
查询操作日志，即审计台“操作日志”页对应接口，记录用户在 JumpServer Web 控制台的增删改查等操作，包含操作用户、动作类型、资源类型、资源名称、来源 IP 与操作时间。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| action | 类型：String，按操作动作过滤 | view（查看）/ update（更新）/ delete（删除）/ create（创建）/ export（导出）/ download（下载）/ connect（连接）/ login（登录）/ change_password（改密）/ accept（接受）/ review（审核）/ notice（通知）/ reject（拒绝）/ approve（批准）/ close（关闭）/ finished（完成） |
| days | 类型：number，最近天数 | 如 `7` 表示最近 7 天内的日志 |
| days__lt | 类型：number，天数上界（早于最近 N 天） | - |
| user | 类型：String，按操作用户过滤 | - |
| resource | 类型：String，按资源名称过滤 | - |
| resource_type | 类型：String，按资源类型过滤 | 如 `资产授权`、`用户` 等页面显示的资源类型名称 |
| remote_addr | 类型：String，按操作来源 IP 过滤 | - |
| search | 类型：String，搜索词（用户、资源等模糊匹配） | - |
| order | 类型：String，排序字段 | 如 `datetime`（升序）/ `-datetime`（降序） |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，操作日志列表 | 列表元素为操作日志对象(见下) |
| id | 类型：string，日志ID | UUID |
| user | 类型：string，操作用户 | 格式如 `名称(用户名)` |
| action | 类型：object，操作动作 | {"value":"update","label":"更新"} 等 |
| resource_type | 类型：string，资源类型 | 如 `资产授权`、`用户` |
| resource | 类型：string，资源名称 | 被操作对象的显示名称 |
| remote_addr | 类型：string，操作来源IP | 可为 null |
| org_id | 类型：string，组织ID |  |
| org_name | 类型：string，组织名称 |  |
| datetime | 类型：string(date-time)，操作时间 |  |

- **响应示例：**

> 示例为示意数据，字段以在线 /api/docs 为准。

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "6f2b8a1c-9d3e-4f5a-b7c8-0a1b2c3d4e5f",
            "user": "张三(zhangsan)",
            "action": {
                "value": "update",
                "label": "更新"
            },
            "resource_type": "资产授权",
            "resource": "运维组-生产服务器授权",
            "remote_addr": "10.1.240.254",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "org_name": "Default",
            "datetime": "2026/07/21 14:32:08 +0800"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/operate-logs/?offset=0&limit=15&order=-datetime' \
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

def get_operate_logs():
    url = f"{API_URL}/api/v1/audits/operate-logs/"
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
        "order": "-datetime",
        "limit": 15,
        "offset": 0
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API 请求失败:{e}")
        return None

if __name__ == "__main__":
    result = get_operate_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：安全审计发现某条资产授权规则的资产范围被扩大，需要追溯这条授权规则是谁在何时修改的：按资源类型“资产授权”过滤更新动作，并以规则名称作为搜索词，按时间倒序查看修改记录中的操作用户与来源 IP。

```sh
curl -X GET 'https://localhost/api/v1/audits/operate-logs/?action=update&resource_type=资产授权&search=运维组-生产服务器授权&order=-datetime&limit=20' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：审计日志对接 SIEM 平台](../examples/siem_export.md)
