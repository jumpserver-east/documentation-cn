## /api/v1/terminal/sessions/

### GET

- **描述：**
查询会话记录（资产会话审计），支持按用户、资产、协议、登录来源、时间范围（最近天数）等条件过滤

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 / 备注 |
| --- | --- | --- |
| user_id | 类型：string，用户 ID | UUID |
| asset_id | 类型：string，资产 ID | UUID |
| protocol | 类型：string，协议 | 如 ssh / rdp / sftp / mysql 等 |
| login_from | 类型：string，登录来源 | ST=SSH Terminal / RT=RDP Terminal / WT=Web Terminal / DT=DB Terminal / VT=VNC Terminal |
| is_finished | 类型：boolean，会话是否已结束 | true / false；false 可筛选在线会话 |
| days | 类型：number，最近 N 天 | 如 days=7 表示只返回最近 7 天内的会话 |
| search | 类型：string，搜索词 | 支持用户名、资产等模糊搜索 |
| order | 类型：string，排序字段 | 如 `-date_start` 按开始时间倒序 |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

> 注：该接口不支持 `date_from` / `date_to` 精确时间段参数（V4 之前的旧资料有误），如需按精确时间段筛选，可先用 `days` 缩小范围，再在客户端按返回的 `date_start` 字段过滤。

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，会话数据列表 | 列表元素为会话对象(见下) |
| id | 类型：string，会话 ID | UUID |
| user | 类型：string，用户 | 格式为 `名称(用户名)` |
| user_id | 类型：string，用户 ID | UUID |
| asset | 类型：string，资产 | 格式为 `名称(地址)` |
| asset_id | 类型：string，资产 ID | UUID |
| account | 类型：string，账号 | 格式为 `名称(用户名)` |
| account_id | 类型：string，账号 ID | UUID |
| protocol | 类型：string，协议 | ssh / rdp / sftp 等 |
| type | 类型：object，会话类型 | 形如 {"value":"sftp","label":"SFTP"} |
| login_from | 类型：object，登录来源 | 形如 {"value":"WT","label":"Web Terminal"} |
| remote_addr | 类型：string，远端地址 | 用户来源 IP |
| comment | 类型：string，备注 | 可为 null |
| terminal_display | 类型：string，组件显示名称 | 处理该会话的终端组件 |
| terminal | 类型：object，终端组件 | 含 id / name / type（如 koko） |
| is_locked | 类型：boolean，是否被锁定 |  |
| is_success | 类型：boolean，是否连接成功 |  |
| is_finished | 类型：boolean，是否已结束 | false 表示在线会话 |
| has_replay | 类型：boolean，是否有录像 |  |
| has_command | 类型：boolean，是否有命令记录 |  |
| can_replay | 类型：boolean，是否可回放 |  |
| can_join | 类型：boolean，是否可加入（协作） |  |
| can_terminate | 类型：boolean，是否可终断 |  |
| command_amount | 类型：int，命令数量 |  |
| error_reason | 类型：object，错误原因 | 形如 {"value":"replay_unsupported","label":"不支持录像"} |
| org_id | 类型：string，组织 ID | UUID |
| org_name | 类型：string，组织名 |  |
| date_start | 类型：string(date-time)，开始时间 |  |
| date_end | 类型：string(date-time)，结束时间 | 未结束时为 null |

- **返回示例：**

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "8608a7af-f1ee-4f84-bb8a-1384b71914f5",
            "user": "xx(jingyu.qi)",
            "asset": "linux-10.1.12.28(10.1.12.28)",
            "user_id": "d461c2e0-95cd-4ccd-aee8-a5d767560eea",
            "asset_id": "4bdae07e-c214-4a12-a9db-be8146219bc8",
            "account": "root(root)",
            "account_id": "cbc45cdb-0e4e-465a-ae73-0b5978240f86",
            "protocol": "sftp",
            "type": {
                "value": "sftp",
                "label": "SFTP"
            },
            "login_from": {
                "value": "WT",
                "label": "Web Terminal"
            },
            "remote_addr": "10.1.240.254",
            "comment": null,
            "terminal_display": "[KoKo]-jms1-22fcb8289055",
            "is_locked": false,
            "command_amount": 0,
            "error_reason": {
                "value": "replay_unsupported",
                "label": "不支持录像"
            },
            "terminal": {
                "id": "82e34994-a778-47d7-ad3f-4ee5e9385cf7",
                "name": "[KoKo]-jms1-22fcb8289055",
                "type": "koko"
            },
            "org_id": "b495b355-b872-48a1-a8ab-615a9be7d49e",
            "org_name": "JS",
            "is_success": true,
            "is_finished": true,
            "has_replay": false,
            "has_command": false,
            "can_replay": false,
            "can_join": false,
            "can_terminate": false,
            "date_start": "2024/01/21 16:47:05 +0800",
            "date_end": "2024/01/21 16:47:44 +0800"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?days=7&is_finished=true&offset=0&limit=15' \
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

def get_sessions():
    url = f"{API_URL}/api/v1/terminal/sessions/"
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
        "days": 7,
        "is_finished": "true",
        "offset": 0,
        "limit": 15
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
    result = get_sessions()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：季度安全审计中，审计员需要拉取最近 7 天内通过 Web Terminal 访问核心数据库资产、且已结束的会话清单，核查是否存在非工作时间的异常访问。

```sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?days=7&login_from=WT&asset_id=4bdae07e-c214-4a12-a9db-be8146219bc8&is_finished=true&order=-date_start&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统一键审计](../examples/one_click_audit.md)

## /api/v1/terminal/sessions/{id}/replay/

### GET

- **描述：**
获取指定会话的录像信息（返回录像文件的下载地址）。如需直接下载录像文件，可访问 `/api/v1/terminal/sessions/{id}/replay/download/` 接口获取文件流。只有 `has_replay` 为 true 的会话才有录像可取

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 响应体为 JSON 格式（download 接口返回文件流） |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string(UUID)，会话 ID（可从会话记录列表接口的返回中获取） | 是 |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| file | 类型：string(uri)，录像文件下载地址 | 会话无录像（如 has_replay=false）时无法获取 |

> 注：`/replay/` 返回录像文件的地址信息；`/replay/download/` 直接返回录像文件流，适合脚本落盘保存后离线取证或导入回放工具。

- **请求示例**

**CURL**

``` sh
# 获取录像信息
curl -X GET 'https://localhost/api/v1/terminal/sessions/SESSION_ID/replay/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# 下载录像文件（保存到本地）
curl -X GET 'https://localhost/api/v1/terminal/sessions/SESSION_ID/replay/download/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -o 'SESSION_ID.replay.gz'
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

def build_auth_headers():
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
    return auth, headers

def get_replay_info():
    url = f"{API_URL}/api/v1/terminal/sessions/{SESSION_ID}/replay/"
    auth, headers = build_auth_headers()

    try:
        response = requests.get(
            url, auth = auth, headers = headers
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API 请求失败:{e}")
        return None

def download_replay(save_path):
    url = f"{API_URL}/api/v1/terminal/sessions/{SESSION_ID}/replay/download/"
    auth, headers = build_auth_headers()

    try:
        response = requests.get(
            url, auth = auth, headers = headers, stream = True
        )
        response.raise_for_status()
        with open(save_path, "wb") as f:
            for chunk in response.iter_content(chunk_size = 8192):
                f.write(chunk)
        print(f"录像已保存: {save_path}")
    except requests.RequestException as e:
        print(f"API 请求失败:{e}")

if __name__ == "__main__":
    info = get_replay_info()
    print(json.dumps(info, indent = 2, ensure_ascii = False))
    download_replay(f"{SESSION_ID}.replay.gz")
```

- **使用案例：**

场景：审计中发现某会话执行了高危命令，审计员根据会话 ID 下载该会话的录像文件，归档到取证目录用于离线回放分析。

```sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/8608a7af-f1ee-4f84-bb8a-1384b71914f5/replay/download/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -o '/data/audit/replays/8608a7af.replay.gz'
```

> 完整集成场景可参考：[实战案例：外部系统一键审计](../examples/one_click_audit.md)
