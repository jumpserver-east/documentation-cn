# 实战案例：生成用户免密直连资产的 URL

## 场景说明

企业内部的运维平台或 CMDB 平台通常希望把 JumpServer 作为底层会话网关内嵌进自己的页面：用户在外部平台点击某台资产，即自动以本人身份打开 JumpServer 的 Web 终端会话，全程无需再手动登录 JumpServer。本案例通过管理员身份的 API 调用，先替目标用户创建资产访问 Token，再为该用户生成一次性免密登录 URL，外部平台只需 `window.open` 这个 URL，即可完成"免密登录 + 直连资产"两步动作。

## 前置条件

1. **开启 SSO 认证功能**：修改 `/opt/jumpserver/config/config.txt` 配置文件，增加下列两行，并执行 `jmsctl restart` 重启服务生效：

    ```text
    AUTH_SSO=True
    AUTH_SSO_AUTHKEY_TTL=900
    ```

    其中 `AUTH_SSO_AUTHKEY_TTL` 为免密登录 authkey 的有效期（秒）。

2. **管理员身份的 API 凭据**：以下接口需要管理员权限，请准备管理员的 Bearer Token，或在个人信息页创建的 API Key（AccessKey ID / Secret，配合 HTTP Signature 签名认证使用）。
3. **目标用户已被授权目标资产**：用户必须已通过资产授权规则获得该资产及对应账号的连接权限，否则无法创建资产访问 Token。
4. 已知目标用户的用户名（如 `zhangsan`）和目标资产的名称（如 `web-server-01`），用于查询各自的 ID。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/users/users/?username=<用户名>` | 按用户名查询用户，取用户 ID |
| GET | `/api/v1/assets/assets/?name=<资产名>` | 按名称查询资产，取资产 ID |
| POST | `/api/v1/authentication/super-connection-token/` | 替用户创建资产访问 Token（超级连接 Token） |
| POST | `/api/v1/authentication/sso/login-url/` | 为用户生成一次性免密登录 URL |

> 各接口字段级明细可在你的 JumpServer 在线 API 文档 `https://<JumpServer地址>/api/docs` 中查看。

## 操作流程

以下示例中的请求头统一为：

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 管理员的认证 Token，格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，示例为默认组织 Default，留空则默认为 Default 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

### 第一步：查询用户 ID

按用户名精确过滤用户列表，从返回结果中取 `id` 字段（UUID），后续创建 Token 时作为 `user` 参数使用。

```sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第二步：查询资产 ID

按资产名称过滤资产列表，从返回结果中取 `id` 字段（UUID），后续作为 `asset` 参数使用。

```sh
curl -X GET 'https://localhost/api/v1/assets/assets/?name=web-server-01' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第三步：替用户创建资产访问 Token

调用 `POST /api/v1/authentication/super-connection-token/`，以管理员身份替目标用户创建一个资产访问 Token。

请求体参数说明：

| 参数名 | 类型 | 描述 | 是否必选 | 备注 |
| --- | --- | --- | --- | --- |
| user | String | 用户 ID | 是 | 第一步查询到的用户 UUID |
| asset | String | 资产 ID | 是 | 第二步查询到的资产 UUID |
| account | String | 账号 | 是 | 托管账号填账号用户名（如 `root`）；手动输入账号填 `@INPUT` |
| protocol | String | 连接协议 | 否 | 不传默认为 `ssh`；如 `rdp`，需为该资产已启用的协议 |
| connect_method | String | 连接方式 | 是 | `web_cli`（字符 Web 终端）、`web_gui`（图形 Web 终端）、`ssh_client`、`mstsc` |
| connect_options | Object | 连接参数 | 否 | 如 `{"resolution": "1920x1080"}` 设置图形会话分辨率 |
| input_username | String | 手动输入的账号用户名 | 否 | 仅 `account` 为 `@INPUT` 时使用 |
| input_secret | String | 手动输入的账号密码 | 否 | 仅 `account` 为 `@INPUT` 时使用 |

```sh
curl -X POST 'https://localhost/api/v1/authentication/super-connection-token/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "user": "53966f1e-ffbc-4f4c-8a10-a6bc70000a98",
            "asset": "fb0c515b-ce80-4391-80b2-5440e5de8133",
            "account": "root",
            "protocol": "ssh",
            "connect_method": "web_cli",
            "connect_options": {
                "resolution": "1920x1080"
            }
        }'
```

返回示例（节选）：

```json
{
    "id": "17a52559-fb2e-4262-8887-fd58020aaa5c",
    "value": "IgJ6aO6wo7GTyAIB",
    "user": {"id": "53966f1e-ffbc-4f4c-8a10-a6bc70000a98", "name": "zhangsan"},
    "asset": {"id": "fb0c515b-ce80-4391-80b2-5440e5de8133", "name": "web-server-01"},
    "account": "root",
    "protocol": "ssh",
    "connect_method": "web_cli",
    "expire_time": 299,
    "is_expired": false,
    "date_expired": "2026/07/22 20:27:53 +0800"
}
```

其中 `id` 就是下一步要用到的 `token_id`；`expire_time` 为 Token 剩余有效期（秒），请在有效期内完成后续步骤。

### 第四步：为用户生成免密登录 URL

调用 `POST /api/v1/authentication/sso/login-url/`，为目标用户生成一次性免密登录 URL。

请求体参数说明：

| 参数名 | 类型 | 描述 | 是否必选 | 备注 |
| --- | --- | --- | --- | --- |
| username | String | JumpServer 用户名 | 是 | 如 `zhangsan`。**注意字段名是 `username`，不是 `user`**（部分旧资料写作 `user`，是错误的） |
| next | String | 登录成功后跳转的地址 | 否 | 不传则登录后进入 JumpServer 默认首页；本案例必须传入，取值见下表 |

`next` 参数常用取值：

| 取值 | 说明 |
| --- | --- |
| `/koko/connect/?token={token_id}` | 内嵌打开字符 Web 终端（SSH/Telnet 等，KoKo 组件），`token_id` 为第三步返回的 `id` |
| `/lion/connect/?token={token_id}` | 内嵌打开图形 Web 终端（RDP/VNC 等，Lion 组件），`token_id` 为第三步返回的 `id` |
| `/luna/?login_to={asset_id}&type=asset` | 打开 Luna 完整 Web 终端页面并定位到指定资产，`asset_id` 为资产 UUID |

```sh
curl -X POST 'https://localhost/api/v1/authentication/sso/login-url/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
            "username": "zhangsan",
            "next": "/koko/connect/?token=17a52559-fb2e-4262-8887-fd58020aaa5c"
        }'
```

返回示例：

```json
{
    "login_url": "https://localhost/api/v1/authentication/sso/login/?authkey=27fe6e65-e1f3-4b9d-9a9a-5aeecad8cf78&next=%2Fkoko%2Fconnect%2F%3Ftoken%3D17a52559-fb2e-4262-8887-fd58020aaa5c"
}
```

### 第五步：外部平台打开 URL

外部平台前端直接 `window.open(login_url)`（或新标签页跳转）即可：浏览器访问该 URL 时自动完成目标用户的免密登录，随后跳转到 `next` 指定的 Web 终端页面，直连目标资产。`login_url` 中的 `authkey` 为一次性凭证，且受 `AUTH_SSO_AUTHKEY_TTL` 有效期限制，应即取即用，不要缓存。

## 完整示例代码

以下 Python 脚本串起完整流程：查用户 ID → 查资产 ID → 创建资产访问 Token → 生成免密登录 URL。认证方式使用 API Key（AK/SK）+ HTTP Signature 签名（依赖 `requests`、`httpsig` 两个包：`pip install requests httpsig`）。

```python
# 生成用户免密直连资产的 URL —— 完整示例
# 流程：查用户 ID -> 查资产 ID -> 创建资产访问 Token -> 生成免密登录 URL

import json
import sys
from datetime import datetime

import requests
from httpsig.requests_auth import HTTPSignatureAuth

API_URL    = "https://localhost"
KEY_ID     = "your id"          # 管理员 API Key 的 AccessKey ID
KEY_SECRET = "your secret"      # 管理员 API Key 的 AccessKey Secret
ORG_ID     = "00000000-0000-0000-0000-000000000002"  # 组织 ID，默认组织 Default

USERNAME       = "zhangsan"        # 需要免密直连资产的 JumpServer 用户名
ASSET_NAME     = "web-server-01"   # 目标资产名称
ACCOUNT        = "root"            # 资产上的托管账号用户名；手动输入账号填 "@INPUT"
PROTOCOL       = "ssh"             # 连接协议
CONNECT_METHOD = "web_cli"         # 字符协议用 web_cli（koko）；图形协议用 web_gui（lion）


def api_request(method, path, params=None, payload=None):
    """带签名认证的通用请求封装，请求失败时抛出异常"""
    url = f"{API_URL}{path}"
    headers = {
        "Accept": "application/json",
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime("%a, %d %b %Y %H:%M:%S GMT"),
    }
    auth = HTTPSignatureAuth(
        key_id=KEY_ID, secret=KEY_SECRET,
        algorithm="hmac-sha256",
        headers=['(request-target)', 'accept', 'date'],
    )
    response = requests.request(
        method, url, auth=auth, headers=headers,
        params=params,
        data=json.dumps(payload) if payload is not None else None,
    )
    response.raise_for_status()
    return response.json()


def get_user_id(username):
    """第一步：按用户名查询用户 ID"""
    data = api_request("GET", "/api/v1/users/users/", params={"username": username})
    users = data["results"] if isinstance(data, dict) else data
    if not users:
        raise ValueError(f"未找到用户: {username}")
    return users[0]["id"]


def get_asset_id(asset_name):
    """第二步：按名称查询资产 ID"""
    data = api_request("GET", "/api/v1/assets/assets/", params={"name": asset_name})
    assets = data["results"] if isinstance(data, dict) else data
    if not assets:
        raise ValueError(f"未找到资产: {asset_name}")
    return assets[0]["id"]


def create_connection_token(user_id, asset_id):
    """第三步：替用户创建资产访问 Token，返回 token_id"""
    payload = {
        "user": user_id,
        "asset": asset_id,
        "account": ACCOUNT,
        "protocol": PROTOCOL,
        "connect_method": CONNECT_METHOD,
        "connect_options": {"resolution": "1920x1080"},
        # 手动输入账号场景（ACCOUNT = "@INPUT"）时补充以下两个字段：
        # "input_username": "root",
        # "input_secret": "<账号密码>",
    }
    token = api_request(
        "POST", "/api/v1/authentication/super-connection-token/", payload=payload
    )
    print(f"Token 创建成功, id={token['id']}, 剩余有效期 {token['expire_time']} 秒")
    return token["id"]


def create_login_url(username, next_url):
    """第四步：为用户生成一次性免密登录 URL。注意字段名是 username，不是 user"""
    payload = {"username": username, "next": next_url}
    data = api_request(
        "POST", "/api/v1/authentication/sso/login-url/", payload=payload
    )
    return data["login_url"]


def main():
    try:
        user_id = get_user_id(USERNAME)
        print(f"用户 {USERNAME} 的 ID: {user_id}")

        asset_id = get_asset_id(ASSET_NAME)
        print(f"资产 {ASSET_NAME} 的 ID: {asset_id}")

        token_id = create_connection_token(user_id, asset_id)

        # 字符协议走 koko 组件页；图形协议（web_gui）请改为 /lion/connect/
        next_url = f"/koko/connect/?token={token_id}"
        login_url = create_login_url(USERNAME, next_url)

        print("免密直连 URL（外部平台 window.open 即可打开）:")
        print(login_url)
    except requests.HTTPError as e:
        print(f"API 请求失败: {e}, 响应内容: {e.response.text}")
        sys.exit(1)
    except (requests.RequestException, ValueError, KeyError) as e:
        print(f"执行出错: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

## 常见问题

**Q1：调用 `/api/v1/authentication/sso/login-url/` 报 400，提示 `username` 字段必填？**

A：该接口的请求体字段是 `username` 和 `next`，部分旧资料中写作 `user`，是错误的。请检查请求体字段名是否为 `username`。

**Q2：打开 `login_url` 后没有免密登录，或提示认证失败？**

A：常见原因有两个：一是没有在 `/opt/jumpserver/config/config.txt` 中配置 `AUTH_SSO=True` 并重启服务；二是 `login_url` 中的 `authkey` 是一次性凭证且受 `AUTH_SSO_AUTHKEY_TTL`（秒）限制，超时或已被使用过就会失效，需重新调用接口生成，不要缓存复用。

**Q3：创建资产访问 Token 时报错，提示账号不存在或没有权限？**

A：`account` 字段的取值要区分场景——托管账号填该资产上的账号用户名（如 `root`），手动输入账号填 `@INPUT` 并同时传入 `input_username`、`input_secret`。此外，目标用户必须已通过资产授权规则获得该资产及对应账号的连接权限，管理员替用户创建 Token 不能绕过授权。

**Q4：生成的 URL 打开后提示 Token 已过期？**

A：资产访问 Token 有效期较短（返回值 `expire_time` 为剩余秒数，默认约 5 分钟）且默认一次性使用（`is_reusable` 为 false）。建议在用户点击资产时才实时调用接口生成 Token 和 `login_url`，即取即用，不要提前批量生成。
