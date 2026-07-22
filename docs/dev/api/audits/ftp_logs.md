## /api/v1/audits/ftp-logs/

### GET

- **描述：**
查询文件传输记录，即审计台“文件传输”页对应的数据，记录用户通过 SFTP、RDP 等方式在资产上进行文件上传、下载等操作的审计日志，包含操作用户、来源地址、资产、账号、文件名、操作动作与成败结果。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| user | 类型：String，按操作用户过滤 | - |
| asset | 类型：String，按资产过滤 | - |
| account | 类型：String，按登录账号过滤 | - |
| filename | 类型：String，按文件名过滤 | - |
| session | 类型：String，按所属会话 ID 过滤 | - |
| search | 类型：String，搜索词（可填用户、资产、文件名等模糊匹配） | - |
| order | 类型：String，排序字段 | 如 `date_start`（升序）/ `-date_start`（降序） |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，文件传输记录列表 | 列表元素为文件传输记录对象(见下) |
| id | 类型：string，记录ID | UUID |
| user | 类型：string，操作用户 | 格式如 `名称(用户名)` |
| remote_addr | 类型：string，来源地址 | 用户客户端 IP，可为 null |
| asset | 类型：string，资产 | 格式如 `名称(地址)` |
| account | 类型：string，登录账号 | 登录资产使用的账号 |
| org_id | 类型：string，组织ID |  |
| operate | 类型：object，操作动作 | {"value":..,"label":..}，如上传/下载/删除等 |
| filename | 类型：string，文件名 | 文件在资产上的路径/名称 |
| date_start | 类型：string(date-time)，操作时间 | 只读字段 |
| is_success | 类型：boolean，是否成功 |  |
| has_file | 类型：boolean，文件是否可下载 | true 表示已留存文件，可在审计台下载 |
| session | 类型：string，所属会话ID | 关联在线/历史会话 |

- **响应示例：**

> 示例为示意数据，字段以在线 /api/docs 为准。

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "5f0f6b52-7a9c-4c3a-9d2e-8b1a2c3d4e5f",
            "user": "张三(zhangsan)",
            "remote_addr": "10.1.240.254",
            "asset": "prod-web-01(192.168.1.10)",
            "account": "root",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "operate": {
                "value": "download",
                "label": "下载"
            },
            "filename": "/data/app/config/app.properties",
            "date_start": "2026-07-20 10:17:20 +0800",
            "is_success": true,
            "has_file": true,
            "session": "9c8b7a6d-5e4f-4a3b-8c1d-0e9f8a7b6c5d"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/ftp-logs/?offset=0&limit=15&order=-date_start' \
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

def get_ftp_logs():
    url = f"{API_URL}/api/v1/audits/ftp-logs/"
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
        "order": "-date_start"
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
    result = get_ftp_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：安全审计发现生产数据疑似外泄，审计员拉取用户 zhangsan 在生产服务器 prod-web-01 上的文件传输记录并按操作时间倒序排列，在客户端根据返回的 `operate` 字段筛出“下载”动作，核对其近期下载了哪些文件。

```sh
curl -X GET 'https://localhost/api/v1/audits/ftp-logs/?user=zhangsan&asset=prod-web-01&order=-date_start&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统一键审计](../examples/one_click_audit.md)
