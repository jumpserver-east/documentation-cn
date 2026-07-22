## /api/v1/terminal/commands/

### GET

- **描述：**
查询会话命令记录。返回用户在资产会话中执行的命令及其输出、风险等级等信息，可按会话、资产、命令内容、风险等级和时间范围过滤。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 备注 |
| --- | --- | --- |
| session_id | 类型：String，会话 ID | 查询指定会话内的命令记录 |
| asset_id | 类型：String(UUID)，资产 ID | 查询指定资产上的命令记录 |
| input | 类型：String，命令输入 | 按命令内容过滤，模糊匹配 |
| risk_level | 类型：Int，命令风险等级 | 可选值：0=接受、4=警告、5=拒绝、6=复核并拒绝、7=复核并接受、8=复核并取消 |
| date_from | 类型：String(date-time)，开始时间 | 例如：2024-01-14T10:07:46.521Z |
| date_to | 类型：String(date-time)，结束时间 | 例如：2024-01-22T15:59:59.000Z |
| search | 类型：String，搜索词 | 通用模糊搜索 |
| limit | 类型：Int，每一页显示条数 | - |
| offset | 类型：Int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，命令记录列表 | 列表元素为命令记录对象(见下) |
| id | 类型：string，命令记录 ID | UUID |
| user | 类型：string，用户名 | 执行命令的用户 |
| asset | 类型：string，资产名 | 形如 `2.7(10.1.12.7)` |
| account | 类型：string，账号名 | 形如 `root(root)` |
| input | 类型：string，命令输入 | 用户实际执行的命令 |
| output | 类型：string，命令输出 | 命令执行的回显内容 |
| session | 类型：string，会话 ID | 命令所属会话 |
| risk_level | 类型：object，命令风险等级 | 形如 {"value":0,"label":"接受"} |
| org_id | 类型：string，组织 ID | UUID |
| timestamp | 类型：int，执行时间戳 | Unix 时间戳（秒） |
| timestamp_display | 类型：string，执行时间 | 形如 `2024/01/19 19:05:40 +0800` |
| remote_addr | 类型：string，远端地址 | 用户来源 IP |

- **返回示例：**

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "user": "mingliang.wang",
            "asset": "2.7(10.1.12.7)",
            "input": "docker ps -a",
            "session": "e37e1476-1a21-424a-909d-9f372c83e36b",
            "risk_level": {
                "value": 0,
                "label": "接受"
            },
            "org_id": "b495b355-b872-48a1-a8ab-615a9be7d49e",
            "id": "0e3e5f7b-4c4f-4711-9cb7-b44d654f7c7f",
            "account": "root(root)",
            "output": "docker ps -a\r\nCONTAINER ID   IMAGE                    COMMAND             CREATED       STATUS                 NAMES\r\n82d85c566fed   jumpserver/lion:v3.7.0   \"./entrypoint.sh\"   4 weeks ago   Up 4 weeks (healthy)   jms_lion",
            "timestamp": 1705662340,
            "timestamp_display": "2024/01/19 19:05:40 +0800",
            "remote_addr": "10.1.240.254"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?session_id=e37e1476-1a21-424a-909d-9f372c83e36b&offset=0&limit=15' \
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
SESSION_ID  = "your session id"

def get_commands():
    url = f"{API_URL}/api/v1/terminal/commands/"
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
        "session_id": SESSION_ID,
        "offset": 0,
        "limit": 15
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
    result = get_commands()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：数据库服务器凌晨出现数据被误删的事故，安全审计员按资产与时间范围拉取包含 `rm` 的命令记录，定位是谁在哪个会话中执行了删除操作。

```sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?asset_id=4bdae07e-c214-4a12-a9db-be8146219bc8&input=rm&date_from=2026-07-21T16:00:00.000Z&date_to=2026-07-22T04:00:00.000Z&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统一键审计](../examples/one_click_audit.md)
