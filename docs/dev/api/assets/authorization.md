## /api/v1/perms/users/{id}/assets/

### GET

- **描述：**
查询指定用户被授权的资产列表

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **路径参数：**

| 参数名 | 类型 | 描述 | 是否必选 |
| --- | --- | --- | --- |
| id | String | 用户 ID，可通过 `/api/v1/users/users/` 接口查询获取 | 是 |

- **查询参数（Query Parameters）：**

| 参数名 | 类型 | 描述 | 是否必选 |
| --- | --- | --- | --- |
| search | String | 搜索词（模糊匹配名称/地址） | 否 |
| name | String | 资产名称 | 否 |
| address | String | IP 地址 | 否 |
| node_id | String | 节点 ID，筛选该节点下的授权资产 | 否 |
| platform | String | 系统平台 | 否 |
| category | String | 类别，如 host | 否 |
| type | String | 类型，如 linux | 否 |
| is_active | Boolean | 激活状态 | 否 |
| limit | Int | 每页显示条数 | 否 |
| offset | Int | 分页偏移量 | 否 |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，授权资产列表 | 列表元素为资产对象(见下) |
| id | 类型：string，资产 ID | UUID |
| name | 类型：string，资产名称 |  |
| address | 类型：string，资产地址 | IP 或域名 |
| platform | 类型：object，系统平台 | {"id":1,"name":"Linux"} 等 |
| nodes | 类型：list，所属节点 |  |
| labels | 类型：list，标签 |  |
| category | 类型：object，类别 | {"value":"host","label":"主机"} 等 |
| type | 类型：object，类型 | {"value":"linux","label":"Linux"} 等 |
| connectivity | 类型：object，可连接性 |  |
| is_active | 类型：boolean，是否激活 |  |
| org_id | 类型：string，组织 ID |  |
| org_name | 类型：string，组织名称 |  |
| date_created | 类型：string(date-time)，创建时间 |  |

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/perms/users/cf7a1f14-0c70-4209-8196-c24cb7ec41a4/assets/?offset=0&limit=15' \
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
USER_ID     = "cf7a1f14-0c70-4209-8196-c24cb7ec41a4"

def get_user_perm_assets(user_id):
    url = f"{API_URL}/api/v1/perms/users/{user_id}/assets/"
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
    result = get_user_perm_assets(USER_ID)
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：运维平台在为工程师张三（zhangsan）开通堡垒机入口前，先核对其当前已被授权的 Linux 生产主机范围，确认仅包含 web 类资产、无越权授权。

```sh
# 张三的用户 ID 可先通过 /api/v1/users/users/?username=zhangsan 查询获取
curl -X GET 'https://localhost/api/v1/perms/users/3f2b9d6c-1a4e-4c58-9d2f-8e7a5b1c0d24/assets/?category=host&type=linux&search=web&is_active=true&limit=20&offset=0' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)
