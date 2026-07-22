## /api/v1/users/users/

### GET

- **描述：**
获取用户列表

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| count | 类型：int，总数 | 分页总记录数 |
| next | 类型：string，下一页链接 | 无更多页为 null |
| previous | 类型：string，上一页链接 | 无上一页为 null |
| results | 类型：list，用户数据列表 | 列表元素为用户对象(见下) |
| id | 类型：string，用户ID | UUID |
| name | 类型：string，显示名称 |  |
| username | 类型：string，用户名 | 登录名 |
| email | 类型：string，邮箱 |  |
| mfa_level | 类型：object，MFA 等级 | {"value":0,"label":"禁用"} 等 |
| source | 类型：object，用户来源 | local/ldap/openid/radius/cas/saml2/oauth2/custom |
| wecom_id | 类型：string，企业微信ID | 绑定时存在 |
| dingtalk_id | 类型：string，钉钉ID | 绑定时存在 |
| feishu_id | 类型：string，飞书ID | 绑定时存在 |
| created_by | 类型：string，创建者 |  |
| updated_by | 类型：string，更新者 |  |
| comment | 类型：string，备注 |  |
| groups | 类型：list，用户组 | 对象或ID列表 |
| system_roles | 类型：list，系统角色 | 默认含“用户”角色 |
| org_roles | 类型：list，组织角色 | 默认含“组织用户” |
| password_strategy | 类型：object，密码策略 | 形如 {"value":"email","label":"..."}；可为 null，默认 value=email（邮件发送重置链接） |
| is_service_account | 类型：boolean，是否组件账号 | true表示系统内部账号 |
| is_valid | 类型：boolean，是否有效 |  |
| is_expired | 类型：boolean，是否到期 |  |
| is_active | 类型：boolean，是否启用 |  |
| is_otp_secret_key_bound | 类型：boolean，是否绑定OTP密钥 |  |
| can_public_key_auth | 类型：boolean，是否允许SSH公钥 |  |
| mfa_enabled | 类型：boolean，是否开启MFA |  |
| need_update_password | 类型：boolean，下次登录需改密 |  |
| mfa_force_enabled | 类型：boolean，是否强制MFA |  |
| is_first_login | 类型：boolean，是否第一次登录 |  |
| login_blocked | 类型：boolean，登录阻止 |  |
| date_expired | 类型：string(date-time)，过期时间 |  |
| date_joined | 类型：string(date-time)，加入时间 |  |
| last_login | 类型：string(date-time)，最后登录时间 |  |
| date_updated | 类型：string(date-time)，更新时间 |  |
| date_password_last_updated | 类型：string(date-time)，密码更新时间 |  |

- **请求示例**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/users/users/?offset=0&limit=15' \
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

def get_user_info():
    url = f"{API_URL}/api/v1/users/users/"
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
    result = get_user_info()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：季度安全审计前，运维需要拉取所有来源为 LDAP 且已被停用的账号清单，核对域账号回收是否遗漏。

```sh
curl -X GET 'https://localhost/api/v1/users/users/?source=ldap&is_active=false&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)

### POST

- **描述：**
创建用户

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，示例为管理员 token；格式固定为 `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，留空则默认为 `Default` 组织 |
| Content-Type | `application/json` | 请求/响应体为 JSON 格式 |

- **请求体参数（Body）：**

| 参数名 | 描述 | 可选值 / 备注 |
| --- | --- | --- |
| name* | 类型：string，名称 / 显示名称 | - |
| username* | 类型：string，用户名（登录名） | - |
| email* | 类型：string，邮箱 | - |
| wechat | 类型：string，微信 | - |
| phone | 类型：string，手机 | - |
| groups | 类型：string[]，用户组 ID 列表 | - |
| password | 类型：string，密码（当 password_strategy=custom 时必填） | - |
| need_update_password | 类型：boolean，是否下次登录需修改密码 | 默认 false；[true,false] |
| public_key | 类型：string，SSH 公钥 | - |
| system_roles | 类型：object[]/string[]，系统角色 | 元素含 pk ID |
| org_roles | 类型：object[]/string[]，组织角色 | 元素含 pk ID |
| password_strategy | 类型：string，密码策略 | email / custom；email=邮件设置密码；请求中传字符串值（如 "email"/"custom"），响应中返回为 object |
| source | 类型：string，用户来源 | 默认 local；可选 local/ldap/ldap_ha/openid/radius/cas/saml2/oauth2/wecom/dingtalk/feishu/lark/slack/custom |
| mfa_level | 类型：integer，MFA 等级 | 0=禁用 1=启用 2=强制 |
| date_expired | 类型：string(date-time)，用户失效时间 | 例如：2023-02-04T00:54:39.000Z |

> 注：带 * 的参数为必填项。
- **返回参数：** （创建成功返回完整用户对象，与 GET 列表中单个元素结构一致）

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| id | 类型：string，用户ID | UUID |
| name | 类型：string，堡垒机用户名称 |  |
| username | 类型：string，堡垒机用户名 | 登录名 |
| email | 类型：string，邮箱 |  |
| wechat | 类型：string，微信 |  |
| phone | 类型：string，电话号码 |  |
| mfa_level | 类型：object，MFA 等级 | {"value":0,"label":"禁用"} 等 |
| source | 类型：object，用户来源 | local/ldap/.../custom |
| wecom_id | 类型：string，企业微信ID | 绑定时存在 |
| dingtalk_id | 类型：string，钉钉ID | 绑定时存在 |
| feishu_id | 类型：string，飞书ID | 绑定时存在 |
| created_by | 类型：string，创建者 |  |
| updated_by | 类型：string，更新者 |  |
| comment | 类型：string，备注 |  |
| groups | 类型：list，用户组 | 对象或ID列表 |
| system_roles | 类型：list，系统角色 | 默认含“用户”角色 |
| org_roles | 类型：list，组织角色 | 默认含“组织用户” |
| password_strategy | 类型：object，密码策略 | 形如 {"value":"email","label":"..."}；可为 null，默认 value=email（邮件发送重置链接） |
| is_service_account | 类型：boolean，是否组件账号 | true 表示系统内部账号 |
| is_valid | 类型：boolean，是否有效 |  |
| is_expired | 类型：boolean，是否到期 |  |
| is_active | 类型：boolean，是否启用 |  |
| is_otp_secret_key_bound | 类型：boolean，是否绑定OTP密钥 |  |
| can_public_key_auth | 类型：boolean，是否允许SSH公钥 |  |
| mfa_enabled | 类型：boolean，是否开启MFA |  |
| need_update_password | 类型：boolean，下次登录需改密 |  |
| mfa_force_enabled | 类型：boolean，是否强制MFA |  |
| is_first_login | 类型：boolean，是否第一次登录 |  |
| login_blocked | 类型：boolean，登录阻止 |  |
| date_expired | 类型：string(date-time)，过期时间 |  |
| date_joined | 类型：string(date-time)，加入时间 |  |
| last_login | 类型：string(date-time)，最后登录时间 |  |
| date_updated | 类型：string(date-time)，更新时间 |  |
| date_password_last_updated | 类型：string(date-time)，密码更新时间 |  |

- **请求示例**

**CURL**

``` sh
curl -X POST 'https://localhost/api/v1/users/users/' \
    -H 'Content-Type:application/json' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ 
            "name": "api_test",
            "username": "api_test",
            "password": "apitest",
            "password_strategy":"custom", 
            "email":"api_test@fit2cloud.com", 
            "mfa_level":0, 
            "source":"local", 
            "system_roles":[ 
                {"pk":"00000000-0000-0000-0000-000000000003"} 
            ], 
            "org_roles":[ 
                {"pk":"00000000-0000-0000-0000-000000000007"} 
            ] 
        }'
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

def create_user():
    url = f"{API_URL}/api/v1/users/users/"
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
        "name": "api_test",
        "username": "api_test",
        "password": "apitest",
        "password_strategy": "custom",
        "email": "api_test@fit2cloud.com",
        "mfa_level": 0,
        "source": "local",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
    }

    try:
        response = requests.post(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("用户创建成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    create_user()
```

- **使用案例：**

场景：新员工张三入职运维组，人事系统触发开号流程：创建本地账号，密码通过邮件发送设置链接，开启 MFA 并按合同期设置账号失效时间。

```sh
curl -X POST 'https://localhost/api/v1/users/users/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "name": "张三",
        "username": "zhangsan",
        "email": "zhangsan@example.com",
        "phone": "13800138000",
        "password_strategy": "email",
        "source": "local",
        "mfa_level": 1,
        "date_expired": "2027-07-21T16:00:00.000Z"
    }'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)

## /api/v1/users/users/{id}/

### GET

- **描述：**
获取用户详情

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | 认证 Token，管理员或具备查看权限的用户 |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，缺省为 Default |
| Content-Type | `application/json` | 响应体为 JSON |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string(UUID)，用户 ID | 是 |

- **返回参数：**（结构与创建/列表单个元素一致）

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| id | 类型：string，用户ID | UUID |
| name | 类型：string，堡垒机用户名称 |  |
| username | 类型：string，堡垒机用户名 | 登录名 |
| email | 类型：string，邮箱 |  |
| wechat | 类型：string，微信 |  |
| phone | 类型：string，电话号码 |  |
| mfa_level | 类型：object，MFA 等级 | {"value":0,"label":"禁用"} 等 |
| source | 类型：object，用户来源 | local/ldap/.../custom |
| wecom_id | 类型：string，企业微信ID | 绑定时存在 |
| dingtalk_id | 类型：string，钉钉ID | 绑定时存在 |
| feishu_id | 类型：string，飞书ID | 绑定时存在 |
| created_by | 类型：string，创建者 |  |
| updated_by | 类型：string，更新者 |  |
| comment | 类型：string，备注 |  |
| groups | 类型：list，用户组 | 对象或ID列表 |
| system_roles | 类型：list，系统角色 | 默认含“用户”角色 |
| org_roles | 类型：list，组织角色 | 默认含“组织用户” |
| password_strategy | 类型：object，密码策略 | 形如 {"value":"email","label":"..."}；可为 null，默认 value=email（邮件发送重置链接） |
| is_service_account | 类型：boolean，是否组件账号 |  |
| is_valid | 类型：boolean，是否有效 |  |
| is_expired | 类型：boolean，是否到期 |  |
| is_active | 类型：boolean，是否启用 |  |
| is_otp_secret_key_bound | 类型：boolean，是否绑定OTP密钥 |  |
| can_public_key_auth | 类型：boolean，是否允许SSH公钥 |  |
| mfa_enabled | 类型：boolean，是否开启MFA |  |
| need_update_password | 类型：boolean，下次登录需改密 |  |
| mfa_force_enabled | 类型：boolean，是否强制MFA |  |
| is_first_login | 类型：boolean，是否第一次登录 |  |
| login_blocked | 类型：boolean，登录阻止 |  |
| date_expired | 类型：string(date-time)，过期时间 |  |
| date_joined | 类型：string(date-time)，加入时间 |  |
| last_login | 类型：string(date-time)，最后登录时间 |  |
| date_updated | 类型：string(date-time)，更新时间 |  |
| date_password_last_updated | 类型：string(date-time)，密码更新时间 |  |

- **请求示例**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Authorization: Bearer <token>' \
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
USER_ID     = "your user id"

def get_user_info():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
    result = get_user_info()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **使用案例：**

场景：用户 zhangsan 提工单反馈无法登录堡垒机，管理员按其用户 ID 查询详情，检查 `is_active`、`is_expired`、`login_blocked` 等字段定位原因。

```sh
curl -X GET 'https://localhost/api/v1/users/users/3f7b9c2e-8d41-4a5b-9c6d-1e2f3a4b5c6d/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)

### PUT

- **描述：**
更新用户

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | 认证 Token，需具备修改权限 |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，缺省 Default |
| Content-Type | `application/json` | 请求/响应体为 JSON |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string(UUID)，用户 ID | 是 |

- **请求体参数（Body）：**

| 参数名 | 描述 | 可选值 / 备注 |
| --- | --- | --- |
| name* | 类型：string，名称 / 显示名称 | - |
| username* | 类型：string，用户名（登录名） | - |
| email* | 类型：string，邮箱 | - |
| wechat | 类型：string，微信 | - |
| phone | 类型：string，手机 | - |
| groups | 类型：string[]，用户组 ID 列表 | - |
| password | 类型：string，密码 | password_strategy=custom 时必填 |
| need_update_password | 类型：boolean，下次登录需改密 | 默认 false |
| public_key | 类型：string，SSH 公钥 | - |
| system_roles | 类型：object[]/string[]，系统角色 | 元素含 pk |
| org_roles | 类型：object[]/string[]，组织角色 | 元素含 pk |
| password_strategy | 类型：string，密码策略 | email / custom；请求中传字符串值（如 "email"/"custom"），响应中返回为 object |
| source | 类型：string，用户来源 | local/ldap/ldap_ha/openid/radius/cas/saml2/oauth2/wecom/dingtalk/feishu/lark/slack/custom |
| mfa_level | 类型：integer，MFA 等级 | 0=禁用 1=启用 2=强制 |
| date_expired | 类型：string(date-time)，用户失效时间 | 2023-02-04T00:54:39.000Z |

> 注：带 * 的参数为必填项。
- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| id | 类型：string，用户ID | UUID |
| name | 类型：string，堡垒机用户名称 |  |
| username | 类型：string，堡垒机用户名 | 登录名 |
| email | 类型：string，邮箱 |  |
| wechat | 类型：string，微信 |  |
| phone | 类型：string，电话号码 |  |
| mfa_level | 类型：object，MFA 等级 | {"value":0,"label":"禁用"} 等 |
| source | 类型：object，用户来源 | local/ldap/.../custom |
| wecom_id | 类型：string，企业微信ID | 绑定时存在 |
| dingtalk_id | 类型：string，钉钉ID | 绑定时存在 |
| feishu_id | 类型：string，飞书ID | 绑定时存在 |
| created_by | 类型：string，创建者 |  |
| updated_by | 类型：string，更新者 |  |
| comment | 类型：string，备注 |  |
| groups | 类型：list，用户组 | 对象或ID列表 |
| system_roles | 类型：list，系统角色 | 默认含“用户”角色 |
| org_roles | 类型：list，组织角色 | 默认含“组织用户” |
| password_strategy | 类型：object，密码策略 | 形如 {"value":"email","label":"..."}；可为 null，默认 value=email（邮件发送重置链接） |
| is_service_account | 类型：boolean，是否组件账号 |  |
| is_valid | 类型：boolean，是否有效 |  |
| is_expired | 类型：boolean，是否到期 |  |
| is_active | 类型：boolean，是否启用 |  |
| is_otp_secret_key_bound | 类型：boolean，是否绑定OTP密钥 |  |
| can_public_key_auth | 类型：boolean，是否允许SSH公钥 |  |
| mfa_enabled | 类型：boolean，是否开启MFA |  |
| need_update_password | 类型：boolean，下次登录需改密 |  |
| mfa_force_enabled | 类型：boolean，是否强制MFA |  |
| is_first_login | 类型：boolean，是否第一次登录 |  |
| login_blocked | 类型：boolean，登录阻止 |  |
| date_expired | 类型：string(date-time)，过期时间 |  |
| date_joined | 类型：string(date-time)，加入时间 |  |
| last_login | 类型：string(date-time)，最后登录时间 |  |
| date_updated | 类型：string(date-time)，更新时间 |  |
| date_password_last_updated | 类型：string(date-time)，密码更新时间 |  |

- **请求示例**

**CURL**
```sh
curl -X PUT 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{
        "name": "new_name",
        "username": "new_username",
        "password_strategy": "email",
        "email": "user@fit2cloud.com",
        "mfa_level": 1,
        "source": "local",
        "date_expired": "2093-02-05T08:28:41.726694Z",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
        }'
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
USER_ID     = "your user id"

def update_user():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
        "name": "api_test",
        "username": "api_test_update",
        "password": "apitest",
        "password_strategy": "custom",
        "email": "api_test@fit2cloud.com",
        "mfa_level": 0,
        "source": "local",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
    }

    try:
        response = requests.put(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("用户更新成功:")
        print(json.dumps(response.json(), indent=2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    update_user()
```

- **使用案例：**

场景：员工李四从测试组转岗到运维组，全量更新其显示名称、邮箱与所属用户组，同时延长账号有效期并保留默认角色。

```sh
curl -X PUT 'https://localhost/api/v1/users/users/9a8b7c6d-5e4f-4a3b-8c1d-0e9f8a7b6c5d/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "name": "李四",
        "username": "lisi",
        "email": "lisi@example.com",
        "groups": ["c1d2e3f4-a5b6-4c7d-8e9f-0a1b2c3d4e5f"],
        "password_strategy": "email",
        "source": "local",
        "mfa_level": 1,
        "date_expired": "2028-12-31T16:00:00.000Z",
        "system_roles": [{"pk": "00000000-0000-0000-0000-000000000003"}],
        "org_roles": [{"pk": "00000000-0000-0000-0000-000000000007"}]
    }'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)

### PATCH

- **描述：**
部分更新用户

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | 认证 Token，需具备修改权限 |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，缺省 Default |
| Content-Type | `application/json` | 请求/响应体为 JSON |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string(UUID)，用户 ID | 是 |

- **请求体（Body 可选字段）：**

| 参数名 | 描述 | 备注 |
| --- | --- | --- |
| name | 类型：string，名称 | 选填 |
| email | 类型：string，邮箱 | 选填 |
| wechat | 类型：string，微信 | 选填 |
| phone | 类型：string，手机 | 选填 |
| groups | 类型：string[]，用户组 ID 列表 | 全量替换该列表 |
| password | 类型：string，新密码 | password_strategy=custom 时使用 |
| need_update_password | 类型：boolean，下次登录需改密 |  |
| public_key | 类型：string，SSH 公钥 |  |
| system_roles | 类型：object[]/string[]，系统角色 | 全量替换 |
| org_roles | 类型：object[]/string[]，组织角色 | 全量替换 |
| password_strategy | 类型：string，密码策略 | email/custom；请求中传字符串值（如 "email"/"custom"），响应中返回为 object |
| source | 类型：string，用户来源 |  |
| mfa_level | 类型：integer，MFA 等级 | 0/1/2 |
| date_expired | 类型：string(date-time)，用户失效时间 |  |

- **返回参数：**

| 字段名称 | 描述 | 备注 |
| --- | --- | --- |
| id | 类型：string，用户ID | UUID |
| name | 类型：string，堡垒机用户名称 |  |
| username | 类型：string，堡垒机用户名 | 登录名 |
| email | 类型：string，邮箱 |  |
| wechat | 类型：string，微信 |  |
| phone | 类型：string，电话号码 |  |
| mfa_level | 类型：object，MFA 等级 | {"value":0,"label":"禁用"} 等 |
| source | 类型：object，用户来源 | local/ldap/.../custom |
| wecom_id | 类型：string，企业微信ID | 绑定时存在 |
| dingtalk_id | 类型：string，钉钉ID | 绑定时存在 |
| feishu_id | 类型：string，飞书ID | 绑定时存在 |
| created_by | 类型：string，创建者 |  |
| updated_by | 类型：string，更新者 |  |
| comment | 类型：string，备注 |  |
| groups | 类型：list，用户组 | 对象或ID列表 |
| system_roles | 类型：list，系统角色 | 默认含“用户”角色 |
| org_roles | 类型：list，组织角色 | 默认含“组织用户” |
| password_strategy | 类型：object，密码策略 | 形如 {"value":"email","label":"..."}；可为 null，默认 value=email（邮件发送重置链接） |
| is_service_account | 类型：boolean，是否组件账号 |  |
| is_valid | 类型：boolean，是否有效 |  |
| is_expired | 类型：boolean，是否到期 |  |
| is_active | 类型：boolean，是否启用 |  |
| is_otp_secret_key_bound | 类型：boolean，是否绑定OTP密钥 |  |
| can_public_key_auth | 类型：boolean，是否允许SSH公钥 |  |
| mfa_enabled | 类型：boolean，是否开启MFA |  |
| need_update_password | 类型：boolean，下次登录需改密 |  |
| mfa_force_enabled | 类型：boolean，是否强制MFA |  |
| is_first_login | 类型：boolean，是否第一次登录 |  |
| login_blocked | 类型：boolean，登录阻止 |  |
| date_expired | 类型：string(date-time)，过期时间 |  |
| date_joined | 类型：string(date-time)，加入时间 |  |
| last_login | 类型：string(date-time)，最后登录时间 |  |
| date_updated | 类型：string(date-time)，更新时间 |  |
| date_password_last_updated | 类型：string(date-time)，密码更新时间 |  |

- **请求示例**

**CURL**
```sh
curl -X PATCH 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -d '{ "name": "partial_update_name" }'
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
USER_ID     = "your user id"

def partial_update_user():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
        "username": "api_test_update"
    }

    try:
        response = requests.patch(
            url, auth = auth, headers = headers,
            data = json.dumps(data)
        )
        response.raise_for_status()
        print("用户更新成功:")
        print(json.dumps(response.json(), indent = 2))
    except Exception as e:
        print(f"错误:{e}")

if __name__ == "__main__":
    partial_update_user()
```

- **使用案例：**

场景：安全整改要求对具备高危权限的账号 zhaoliu 强制启用 MFA，并要求其下次登录时修改密码，其余信息保持不变。

```sh
curl -X PATCH 'https://localhost/api/v1/users/users/7e6d5c4b-3a2f-4e1d-9c8b-7a6f5e4d3c2b/' \
    -H 'Content-Type: application/json' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>' \
    -d '{
        "mfa_level": 2,
        "need_update_password": true
    }'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)

### DELETE

- **描述：**
删除用户

- **请求头（Headers）：**

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer <token>` | 认证 Token，需具备删除权限 |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，缺省 Default |

- **路径参数（Path Params）：**

| 名称 | 说明 | 必填 |
| --- | --- | --- |
| id | 类型：string(UUID)，用户 ID | 是 |

- **返回：** 204 No Content（删除成功无响应体）

- **请求示例**

**CURL**
```sh
curl -X DELETE 'https://localhost/api/v1/users/users/USER_ID/' \
    -H 'Authorization: Bearer <token>' \
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
USER_ID     = "your user id"

def delete_user():
    url = f"{API_URL}/api/v1/users/users/{USER_ID}/"
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
        print(f"用户删除成功: {response.status_code}")
    except Exception as e:
        print(f"API 请求失败:{e}")
        return None

if __name__ == "__main__":
    delete_user()
```

- **使用案例：**

场景：员工王五离职，离职工单终审通过后，由自动化脚本删除其堡垒机账号，完成访问权限回收。

```sh
curl -X DELETE 'https://localhost/api/v1/users/users/5b4a3c2d-1e0f-4d9c-8b7a-6f5e4d3c2b1a/' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：用户生命周期自动化](../examples/user_lifecycle.md)
