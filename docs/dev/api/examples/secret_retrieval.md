# 实战案例：外部脚本免硬编码获取资产账号密码

## 场景说明

运维脚本、定时任务或 CI/CD 流水线在连接服务器、数据库时，往往把密码硬编码在脚本、配置文件或流水线变量里：密码一旦轮转就要到处修改，且极易随代码仓库扩散泄露。本案例演示如何让外部脚本在运行时通过 JumpServer API 实时取密——脚本本地只保留 JumpServer 的认证凭据（通过环境变量注入），不落地任何业务密码；再配合改密计划定期轮转资产账号密码，脚本每次运行拿到的永远是最新密码，实现"密码只存在于 JumpServer"。

## 前置条件

- 已部署 JumpServer V4，运行脚本的机器可访问 JumpServer 的 HTTPS 端口。
- 已获取调用凭据：API Key（AK/SK，Web 页面右上角头像 - API Key 中创建）或 Bearer Token，调用者需具备目标资产账号的查看权限。
- 目标资产及其账号已在 JumpServer 中托管密码（可用账号列表接口的 `has_secret=true` 过滤确认）。
- 方式一（直连取密）默认要求 MFA 认证。如需 API 直接调用，需在 `/opt/jumpserver/config/config.txt` 中增加参数 `SECURITY_VIEW_AUTH_NEED_MFA=False`，并执行 `jmsctl restart` 重启后生效，详见[账号密文查询接口文档](../accounts/accountsecrets.md)。
- 方式二（PAM 集成应用）需要具备创建集成应用的权限，接口明细见[应用集成接口文档](../pam/integration-application.md)。
- Python 完整示例依赖：`pip install requests httpsig`。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/accounts/accounts/` | 按资产查询账号列表，拿到账号 ID |
| GET | `/api/v1/accounts/account-secrets/{id}/` | 按账号 ID 实时获取账号密文（方式一） |
| POST | `/api/v1/accounts/integration-applications/` | 创建 PAM 集成应用（方式二） |
| GET | `/api/v1/accounts/integration-applications/{id}/secret/` | 获取集成应用的应用密钥（方式二） |
| GET | `/api/v1/accounts/integration-applications/account-secret/` | 通过集成应用按资产 + 账号获取密文（方式二） |

> 以上接口的字段级明细，可在你部署环境的在线 API 文档 `https://<JumpServer地址>/api/docs` 中查看。

## 操作流程

### 方式一：直连取密（简单直接，适合受控内网环境）

#### 步骤 1：按资产查询账号列表

调用账号列表接口，用 `asset_id` 限定资产，可叠加 `username`、`has_secret` 等查询参数进一步过滤，从返回结果中拿到账号 ID。

```sh
curl -X GET 'https://localhost/api/v1/accounts/accounts/?asset_id=9266b1f8-f74d-482c-805a-6eed0e099a42&username=root&has_secret=true' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回的每个账号对象包含 `id`、`name`、`username`、`secret_type`、`asset`、`privileged`、`is_active`、`version` 等字段。注意：该接口不会返回 `secret` 明文（`secret` 是只写字段），必须用返回的 `id` 走下一步取密。

#### 步骤 2：按账号 ID 获取密文

用上一步得到的账号 ID 调用账号密文接口，返回中的 `secret` 即密码/密钥明文，`version` 为密文版本（每次改密后递增）。

```sh
curl -X GET 'https://localhost/api/v1/accounts/account-secrets/1e28b088-c86c-41b7-a85b-29e15c7cc8cb/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回示例（节选）：

```json
{
    "id": "1e28b088-c86c-41b7-a85b-29e15c7cc8cb",
    "name": "root",
    "username": "root",
    "secret_type": {"value": "password", "label": "密码"},
    "secret": "Xy3#example-secret",
    "version": 3
}
```

> 若返回 403 或提示需要 MFA，请检查前置条件中的 `SECURITY_VIEW_AUTH_NEED_MFA` 配置是否已生效。

### 方式二：PAM 集成应用取密（生产推荐，权限更收敛）

集成应用相当于给每个外部系统发放一个独立的"应用身份"：只授权它所需的账号范围，并可用 `ip_group` 限制来源 IP。相比直接使用管理员 Token/AK，凭据一旦泄露影响面小得多，适合生产环境长期运行的脚本与流水线。

#### 步骤 3：创建集成应用

指定应用名称、授权账号范围与允许访问的来源 IP 段。

```sh
curl -X POST 'https://localhost/api/v1/accounts/integration-applications/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "cicd-runner",
        "accounts": {"ids": ["1e28b088-c86c-41b7-a85b-29e15c7cc8cb"]},
        "ip_group": ["192.168.1.0/24"],
        "is_active": true,
        "comment": "CI 流水线取密专用"
    }'
```

请求体中 `name`、`accounts` 为必填；`ip_group` 默认 `["*"]`（不限来源），生产环境建议收敛为脚本机器的出口 IP。`accounts` 的内部取值结构请以在线 API 文档为准。返回中记下应用 `id`。

#### 步骤 4：获取集成应用的应用密钥

用应用 ID 调用一次性密钥接口，获得该应用的调用密钥。敏感内容仅在首次调用时返回，请立即妥善保存（如写入 CI 的 Secret 变量），丢失后需重新生成。

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/9b6b5a8e-6a1f-4c2a-9d0e-3f5e8a7b1c2d/secret/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

#### 步骤 5：通过集成应用按资产 + 账号取密

外部脚本使用集成应用身份调用统一取密接口，按资产与账号定位目标密文。

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/account-secret/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

> 该接口用于指定资产与账号的具体查询参数、以及密文返回字段，未收录在本仓库的 swagger 定义中（swagger 中返回结构 `IntegrationAccountSecret` 仅声明 `asset`、`asset_id`、`account`、`account_id`）。实际调用时请以部署环境在线文档 `https://<JumpServer地址>/api/docs` 中 `accounts_integration_applications_account_secret` 的定义为准，或在集成应用详情页查看内置的调用示例（SDK）。

### 配合改密计划实现定期轮转

密码轮转由 JumpServer 改密计划完成：在 Web 控制台「PAM - 账号改密」中创建周期性改密计划（也可通过 API 创建，参考[改密计划接口文档](../pam/changepwd.md)），把资产账号定期改为随机密码。轮转后账号密文 `version` 自动递增，而脚本每次运行都实时取密，无需任何改动即可拿到最新密码——这正是"免硬编码"的收益：改密不再需要同步修改任何脚本或配置。

## 完整示例代码

以下 Python 脚本串起方式一的完整流程：按资产 + 用户名定位账号 → 实时获取密文 → 使用密码连接目标。JumpServer 的 AK/SK 通过环境变量注入，代码与仓库中不出现任何密码明文。

```python
# -*- coding: utf-8 -*-
"""
外部脚本免硬编码获取资产账号密码

流程：查询资产账号列表 -> 定位目标账号 -> 实时获取账号密文
认证信息全部通过环境变量注入，脚本与代码仓库中不落地任何密码明文。

依赖安装：
    pip install requests httpsig

环境变量：
    JMS_URL         JumpServer 访问地址，如 https://localhost
    JMS_KEY_ID      API Key 的 AccessKey ID
    JMS_KEY_SECRET  API Key 的 AccessKey Secret
    JMS_ORG_ID      组织 ID，默认组织为 00000000-0000-0000-0000-000000000002
"""

import os
import sys
from datetime import datetime

import requests
from httpsig.requests_auth import HTTPSignatureAuth

API_URL    = os.environ.get("JMS_URL", "https://localhost")
KEY_ID     = os.environ.get("JMS_KEY_ID")
KEY_SECRET = os.environ.get("JMS_KEY_SECRET")
ORG_ID     = os.environ.get("JMS_ORG_ID", "00000000-0000-0000-0000-000000000002")

ASSET_ID   = "9266b1f8-f74d-482c-805a-6eed0e099a42"   # 目标资产 ID
USERNAME   = "root"                                   # 目标账号用户名


def build_auth():
    """构造 AK/SK 签名认证对象"""
    signature_headers = ['(request-target)', 'accept', 'date']
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = signature_headers
    )


def build_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Accept": "application/json",
        "Content-Type": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }


def get_account_id(asset_id, username):
    """步骤一：按资产 + 用户名查询账号，返回账号 ID"""
    url = f"{API_URL}/api/v1/accounts/accounts/"
    params = {
        "asset_id": asset_id,
        "username": username,
        "has_secret": True
    }
    response = requests.get(
        url, auth = build_auth(), headers = build_headers(),
        params = params
    )
    response.raise_for_status()
    data = response.json()
    # 带分页参数时返回 {count, results}，否则可能直接返回列表，两种都兼容
    results = data.get("results") if isinstance(data, dict) else data
    if not results:
        raise LookupError(f"资产 {asset_id} 下未找到已托管密码的账号 {username}")
    return results[0]["id"]


def get_account_secret(account_id):
    """步骤二：实时获取账号密文（永远是当前最新版本）"""
    url = f"{API_URL}/api/v1/accounts/account-secrets/{account_id}/"
    response = requests.get(
        url, auth = build_auth(), headers = build_headers()
    )
    response.raise_for_status()
    return response.json()


def main():
    if not KEY_ID or not KEY_SECRET:
        print("请先通过环境变量 JMS_KEY_ID / JMS_KEY_SECRET 注入认证信息")
        sys.exit(1)

    try:
        account_id = get_account_id(ASSET_ID, USERNAME)
        secret_info = get_account_secret(account_id)
    except requests.HTTPError as e:
        status = e.response.status_code if e.response is not None else "?"
        if str(status) == "403":
            print("取密被拒绝（403）：请确认调用者权限，以及 config.txt 中是否已"
                  "配置 SECURITY_VIEW_AUTH_NEED_MFA=False 并重启服务")
        else:
            print(f"API 请求失败（HTTP {status}）：{e}")
        sys.exit(2)
    except requests.RequestException as e:
        print(f"网络请求异常：{e}")
        sys.exit(2)
    except LookupError as e:
        print(f"数据错误：{e}")
        sys.exit(3)

    username = secret_info.get("username")
    secret   = secret_info.get("secret")
    version  = secret_info.get("version")
    if not secret:
        print("接口返回成功但密文为空，请确认该账号已在 JumpServer 中托管密码")
        sys.exit(3)

    # 演示：仅打印脱敏信息。切勿在生产日志中输出密码明文！
    print(f"账号: {username}  密文版本: {version}  密码: {secret[:2]}****")

    # 在这里使用取到的密码连接目标资产，例如通过 paramiko 连接 SSH：
    # import paramiko
    # ssh = paramiko.SSHClient()
    # ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # ssh.connect('<资产地址>', 22, username, secret)
    # 密码只存在于本次进程内存中，随进程结束而消失


if __name__ == "__main__":
    main()
```

## 常见问题

**Q1：调用 `/api/v1/accounts/account-secrets/{id}/` 返回 403 或提示需要 MFA 认证？**

A：查看账号密文默认要求 MFA 认证，API 场景无法交互输入 MFA。需在 `/opt/jumpserver/config/config.txt` 中增加 `SECURITY_VIEW_AUTH_NEED_MFA=False` 并执行 `jmsctl restart` 后重试。注意该配置会降低取密门槛，建议仅在受控网络中开启，并确保调用凭据妥善保管。若配置已生效仍 403，请检查调用者是否具备该账号的查看权限、`X-JMS-ORG` 是否指向账号所在组织。

**Q2：直连取密和 PAM 集成应用两种方式怎么选？**

A：直连取密（方式一）实现最简单，但使用的是用户级 Token/AK——凭据泄露即意味着该用户的全部 API 权限泄露，适合内网受控环境、临时排查或小规模脚本。PAM 集成应用（方式二）为每个外部系统发放独立应用身份，只授权所需账号范围，还能用 `ip_group` 限制来源 IP，可随时单独禁用（`is_active=false`）而不影响其他系统，生产环境的长期脚本、CI 流水线推荐使用方式二。

**Q3：为什么 `GET /api/v1/accounts/accounts/` 的返回里没有密码字段？**

A：账号对象中的 `secret` 是只写（writeOnly）字段，仅在创建/更新账号时可提交，任何列表和详情接口都不会返回明文。这是有意设计：取密必须显式调用 `account-secrets` 或集成应用的取密接口，便于权限收敛与审计。因此流程上必须"先查账号 ID、再换取密文"，两步不能合并。

**Q4：改密计划轮转密码后，正在运行的脚本会不会拿到旧密码？**

A：不会。取密接口返回的始终是当前最新版本的密文（`version` 字段随每次改密递增），脚本只要坚持"每次运行时实时取密、不缓存密码"，轮转对它就是透明的。需要注意的是：已经建立的长连接（如连接池中的数据库会话）不受改密影响，但断线重连时必须重新调用取密接口，用旧密码重连会认证失败；因此不要把密码缓存到文件或环境变量中复用。
