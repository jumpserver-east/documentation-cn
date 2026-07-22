## /api/v1/assets/nodes/

### GET
- **描述：**
查询节点

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| search | 类型：String，搜索词，支持节点名搜索 | - |
| limit | 类型：int，每一页显示条数 | - |
| offset | 类型：int，分页偏移量 | - |

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，节点id |  |
| key | 类型：String，键 |  |
| value | 类型：String，值\节点名称 |  |
| org_id | 类型：String，组织 |  |
| name | 类型：String，节点名称（只读） |  |
| full_value | 类型：String，全称 |  |
| org_name | 类型：String，组织名称 |  |

**请求示例**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/?search=/Default/xx公司' \
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
SEARCH_WORD = "your search word"

def search_nodes(keyword):
    url = f"{API_URL}/api/v1/assets/nodes/"
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

    params = {"search": keyword}

    try:
        response = requests.get(
            url, auth = auth, headers = headers,
            params = params
        )
        response.raise_for_status()
        nodes_data = response.json()
        nodes = nodes_data.get("results", [])
        if not nodes:
            print("未找到匹配的资产节点")
        else:
            print(f"查询到 {nodes_data['count']} 个匹配的资产节点：")
            print(json.dumps(nodes_data, indent = 2, ensure_ascii = False))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    search_nodes(SEARCH_WORD)
```
- **响应示例：**

```json
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": "aa2aa3fe-c2c3-49ca-b4bd-d04dc8bf6693",
      "key": "1",
      "value": "DEFAULT",
      "org_id": "00000000-0000-0000-0000-000000000002",
      "name": "DEFAULT",
      "full_value": "/DEFAULT",
      "org_name": "DEFAULT"
    }
  ]
}
```

- **使用案例：**

场景：CMDB 系统与堡垒机做资产结构对账，先按节点名搜索"生产环境"节点确认其是否已存在，并限制每页返回 20 条以便分页遍历。

```sh
curl -X GET 'https://localhost/api/v1/assets/nodes/?search=生产环境&limit=20&offset=0' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)

##  /api/v1/assets/nodes/{id}/children/

### POST
- **描述：**
创建节点

- **请求头（Headers）：**

| 键 (Header)      | 示例值                                       | 说明                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type      | `application/json`                            | 请求/响应体为 JSON 格式           |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string，节点ID | 是 |

- **请求体参数（Body）：**

| 参数名 | 描述 | 可选值 |
| --- | --- | --- |
| value | 类型：String，节点名称；swagger 定义中该字段非必填（nullable），但创建节点时建议传入以指定节点名称 | - |

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String，节点id |  |
| key | 类型：String，键 |  |
| value | 类型：String，值\节点名称 |  |
| org_id | 类型：String，组织 |  |
| name | 类型：String，节点名称（只读） |  |
| full_value | 类型：String，全称 |  |
| org_name | 类型：String，组织名称 |  |

**请求示例**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/assets/nodes/3728f004-99a2-4fca-9577-84d5ffcf9eff/children/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{"value":"test_create_node"}'
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
NODE_ID     = "your node id"
NODE_NAME   = "your node name"

def create_children_nodes(node_id, node_name):
    url = f"{API_URL}/api/v1/assets/nodes/{node_id}/children/"
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
        "value": node_name
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("资产节点创建成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    create_children_nodes(NODE_ID, NODE_NAME)
```

- **使用案例：**

场景：新业务线"电商中台"上线，运维平台在"生产环境"节点（ID 为 `3728f004-99a2-4fca-9577-84d5ffcf9eff`）下自动创建同名子节点，供后续批量纳管该业务线的服务器。

```sh
curl -X POST 'https://localhost/api/v1/assets/nodes/3728f004-99a2-4fca-9577-84d5ffcf9eff/children/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{"value":"电商中台"}'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)

##  /api/v1/assets/nodes/{id}/ 

### DELETE
- **描述：**
删除节点

- **请求头（Headers）：**

| 键 (Header)      | 示例值                                       | 说明                            |
| ---------------- | -------------------------------------------- | ------------------------------- |
| Authorization     | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG         | `00000000-0000-0000-0000-000000000002`         | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type      | `application/json`                            | 请求/响应体为 JSON 格式           |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string，节点ID | 是 |


**请求示例**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/assets/nodes/89c68ef6-7790-4f20-8f8d-fdd76d229b3d/' \
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
NODE_ID     = "your node id"

def delete_assets_nodes(node_id):
    url = f"{API_URL}/api/v1/assets/nodes/{node_id}/"
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
            url, auth = auth, headers = headers
        )
        response.raise_for_status()
        print("资产节点删除成功")
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    delete_assets_nodes(NODE_ID)
```

- **使用案例：**

场景：老旧项目"报表系统"整体下线，其资产已全部迁出，管理员在清理脚本中删除该项目对应的空节点（ID 为 `89c68ef6-7790-4f20-8f8d-fdd76d229b3d`），保持资产树整洁。

```sh
curl -X DELETE 'https://localhost/api/v1/assets/nodes/89c68ef6-7790-4f20-8f8d-fdd76d229b3d/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部系统申请资产并自动授权](../examples/asset_sync_authorize.md)
