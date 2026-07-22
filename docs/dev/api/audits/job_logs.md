## /api/v1/audits/job-logs/

### GET

- **描述：**
查询作业日志，即审计台“作业审计”页对应的数据，记录作业中心执行的命令（Adhoc）、Playbook、文件上传等任务的执行内容、执行人、起止时间与成败结果。

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| creator__name | 类型：String，按创建者（执行人）名称过滤 | - |
| material | 类型：String，按执行内容（命令/脚本）过滤 | - |
| search | 类型：String，搜索词（模糊匹配） | - |
| order | 类型：String，排序字段 | 如 `date_start`（升序）/ `-date_start`（降序） |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，作业日志列表 | 列表元素为作业日志对象(见下) |
| id | 类型：string，日志ID | UUID |
| material | 类型：string，执行内容（命令/脚本） | 可为 null |
| job_type | 类型：string，作业类型 | adhoc（命令）/ playbook（剧本）/ upload_file（文件上传），默认 adhoc |
| time_cost | 类型：int，执行耗时 | 单位：秒 |
| is_finished | 类型：boolean，是否已完成 |  |
| is_success | 类型：boolean，是否执行成功 |  |
| task_id | 类型：string，任务ID | UUID，可为 null |
| creator_name | 类型：string，创建者（执行人） |  |
| org_id | 类型：string，组织ID |  |
| org_name | 类型：string，组织名称 |  |
| date_start | 类型：string(date-time)，开始时间 | 可为 null |
| date_finished | 类型：string(date-time)，结束时间 | 可为 null |
| date_created | 类型：string(date-time)，创建时间 |  |

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
            "material": "df -h",
            "job_type": "adhoc",
            "time_cost": 6,
            "is_finished": true,
            "is_success": true,
            "task_id": "9a8b7c6d-5e4f-4a3b-8c1d-0e9f8a7b6c5d",
            "creator_name": "zhangsan",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "org_name": "Default",
            "date_start": "2026-07-21 22:05:10 +0800",
            "date_finished": "2026-07-21 22:05:16 +0800",
            "date_created": "2026-07-21 22:05:09 +0800"
        }
    ]
}
```

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/job-logs/?offset=0&limit=15&order=-date_start' \
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

def get_job_logs():
    url = f"{API_URL}/api/v1/audits/job-logs/"
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
    result = get_job_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：变更窗口结束后，运维值班人员按开始时间倒序拉取当晚由 zhangsan 批量执行的作业日志，在客户端根据 `date_start` 截取变更窗口时段的数据，逐条检查 `is_finished` 与 `is_success` 字段，确认批量作业是否全部成功。

```sh
curl -X GET 'https://localhost/api/v1/audits/job-logs/?creator__name=zhangsan&order=-date_start&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统一键审计](../examples/one_click_audit.md)
