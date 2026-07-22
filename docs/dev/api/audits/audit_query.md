# 1. 会话记录

 

#### 1.1  查询会话记录

 

**(1)**   **API** **地址**

| 请求方式 | Request URL                | 备注             |
| -------- | -------------------------- | ---------------- |
| GET      | /api/v1/terminal/sessions/ | 查询会话记录接口 |

**(2)**   **Request Header**

| 键            | 值                                             | 备注                                                         |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Authorization | Token $TOKEN | 管理员token信息                                              |
| X-JMS-ORG     | 00000000-0000-0000-0000-000000000002           | 00000000-0000-0000-0000-000000000002为组织ID，此id号为默认组织：**Default**，留空则默认为 **Default** 组织。 |
| accept        | application/json,  text/plain, */*             | 数据格式                                                     |

**(3)**   **Query Parameters**

| 参数名      | 类型   | 描述           | 是否必选 | 默认值               |
| ----------- | ------ | -------------- | -------- | -------------------- |
| user_id     | String | 用户id         | 否       |                      |
| asset_id    | String | 资产id         | 否       |                      |
| user        | String | 用户名称       | 否       |                      |
| asset       | String | 资产名称       | 否       |                      |
| account     | String | 资产账号       | 否       |                      |
| order       | String | 排序           | 否       |                      |
| date_from   | String | 开始时间       | 是       |                      |
| date_to     | String | 结束时间       | 是       |                      |
| offset      | String | 每一页显示条数 | 是       |                      |
| limit       | String | 分页偏移量     | 是       |                      |
| is_finished | String | 是否结束       | 是       | 0：未完成；1：已完成 |

**(4)**   **请求示例**

```
curl --location --request GET 'https://example.jumpserver.org/api/v1/terminal/sessions/?is_finished=1&date_from=2026-05-06T03%3A30%3A14.860Z&date_to=2026-05-14T15%3A59%3A59.000Z&offset=0&limit=15&display=1&draw=1' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: application/json' \
--header 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'
```

**(5)**   **返回示例**

```
{
    "count": 9,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "59a412c7-61c6-4ce6-bae9-f695e3f52c98",
            "user": "other_admin(other_admin)",
            "asset": "JMS-Itself(10.1.14.13)",
            "user_id": "1871934c-6486-4817-bdc5-27c2ba7307f7",
            "asset_id": "de51ad80-5121-4cf6-908b-859bbc2ef84b",
            "account": "root(root)",
            "account_id": "20edc9c7-ae40-4f1f-955c-91de1eb62afc",
            "protocol": "ssh",
            "type": {
                "value": "normal",
                "label": "Normal"
            },
            "login_from": {
                "value": "WT",
                "label": "Web Terminal"
            },
            "remote_addr": "10.1.10.35",
            "duration": "3:43:11",
            "terminal_display": "[KoKo]-jumpserver-v4-jms_koko-UMcpvoM",
            "is_locked": false,
            "command_amount": 0,
            "error_reason": {
                "value": "",
                "label": ""
            },
            "replay_size": 993,
            "terminal": {
                "id": "023b2a7b-4615-43f0-8e49-4b7b0d872ffa",
                "name": "[KoKo]-jumpserver-v4-jms_koko-UMcpvoM",
                "type": "koko"
            },
            "org_id": "00000000-0000-0000-0000-000000000002",
            "org_name": "DEFAULT",
            "is_success": true,
            "is_finished": true,
            "has_replay": true,
            "has_command": false,
            "can_replay": true,
            "can_join": false,
            "can_terminate": false,
            "date_start": "2026/05/09 14:26:58 +0800",
            "date_end": "2026/05/09 18:10:09 +0800",
            "comment": null
        }
     ]
}
```

**(6)**   **返回示例参数说明**

| 字段名称       | 数据类型 | 字段描述   | 备注 |
| -------------- | -------- | ---------- | ---- |
| count          | Int      | 总数       |      |
| next           | String   | 下一页链接 |      |
| previous       | String   | 上一页链接 |      |
| results        | List     | 数据       |      |
| id             | String   | id         |      |
| user           | String   | 用户       |      |
| user_id        | String   | 用户id     |      |
| asset          | String   | 资产名     |      |
| asset_id       | String   | 资产id     |      |
| account        | String   | 账号名     |      |
| account_id     | String   | 账号id     |      |
| protocol       | String   | 协议       |      |
| org_name       | String   | 组织名     |      |
| can_replay     | Boolean  | 是否可回放 |      |
| has_command    | Boolean  | 是否有命令 |      |
| command_amount | Boolean  | 命令数量   |      |



#### **1.2 获取录像元数据接口**

**(1)**   **API** **地址**

| 请求方式 | Request URL                                 | 备注           |
| -------- | ------------------------------------------- | -------------- |
| GET      | /api/v1/terminal/sessions/#{会话id}/replay/ | 查询录像元数据 |

**(2)**   **Request Header**

| 键            | 值                                             | 备注                     |
| ------------- | ---------------------------------------------- | ------------------------ |
| Authorization | Token $TOKEN | 管理员token信息          |
| X-JMS-ORG     | ba5df883-3a45-450f-8778-320995e0e282           | 组织ID                   |
| accept        | ***\*/\****                                    | 告诉服务器接收的数据格式 |

**(3)**   **Query Parameters**

此为路径参数，路径参数为会话id

**(4)**   **请求示例**

  

```
curl --location --request GET 'https://example.jumpserver.org/api/v1/terminal/sessions/f6884eda-b1c1-408e-83b2-fc1234cde52c/replay/' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7' \
--header 'X-JMS-ORG: ba5df883-3a45-450f-8778-320995e0e282' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'
```

**(5)**   **返回示例**

```
{
    "status": "ok",
    "start_time": 1778724933.0611722,
    "key": "GET_e137ea987929ad6a3c051c50c0dfcc37_b8cdae68-3cfe-4f13-90d7-fbbaa811be17",
    "resp": {
        "data": {
            "type": "asciicast",
            "src": "/media/replay/2026-05-12/f6884eda-b1c1-408e-83b2-fc1234cde52c.cast.gz",
            "user": "yuanzhenjie(yuanzhenjie)",
            "asset": "jumpserver02-10.1.13.46(10.1.13.46)",
            "account": "root(root)",
            "date_start": "2026-05-12T10:24:27Z",
            "date_end": "2026-05-12T10:28:29Z",
            "download_url": "/api/v1/terminal/sessions/f6884eda-b1c1-408e-83b2-fc1234cde52c/replay/download/"
        },
        "status": 200
    }
}
```

**(6)**   **返回示例参数说明**

| 字段                              | 含义                         | 作用                                                         |
| --------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| `status: "ok"`                    | 接口调用状态                 | 表示元数据获取成功                                           |
| `resp.data.type: "asciicast"`     | 录像文件格式                 | 说明这是 `asciicast` 格式的终端录像（`.cast.gz` 文件），是 JumpServer 常用的回放格式 |
| `resp.data.src`                   | 录像文件在服务器上的存储路径 | 内部路径，无法直接通过浏览器访问，仅用于服务器定位文件       |
| `resp.data.download_url`          | 录像文件的下载接口地址       | 就是你之前找到的 `/api/v1/terminal/sessions/xxx/replay/download/`，播放器 / 下载工具通过这个地址获取完整录像 |
| `resp.data.user/asset/date_start` | 会话基础信息                 | 播放器界面显示的会话详情，和你会话列表里的信息完全一致       |



#### **1.3 会话录像下载**

**(1)**   **API** **地址**



| 请求方式 | Request URL                                          | 备注                 |
| -------- | ---------------------------------------------------- | -------------------- |
| GET      | /api/v1/terminal/sessions/#{会话id}/replay/download/ | 查询用户登录日志记录 |

**(2)**   **Request Header**

| 键            | 值                                                           | 备注                     |
| ------------- | ------------------------------------------------------------ | ------------------------ |
| Authorization | Token $TOKEN               | 管理员token信息          |
| X-JMS-ORG     | ba5df883-3a45-450f-8778-320995e0e282                         | 组织ID                   |
| accept        | text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7 | 告诉服务器接收的数据格式 |

**(3)**   **Query Parameters**

此为路径参数，为会话id

**(4)**   **请求示例**



```
curl --location --request GET 'https://example.jumpserver.org/api/v1/terminal/sessions/f6884eda-b1c1-408e-83b2-fc1234cde52c/replay/download/' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7' \
--header 'X-JMS-ORG: ba5df883-3a45-450f-8778-320995e0e282' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'
```

**(5)**   **返回示例**

文件：application/octet-stream

**(6)**   **返回示例参数说明**

返回对应录像文件



# 2.命令记录



#### **2.1  获取命令记录**

**(1)**   **API** **地址**

| 请求方式 | Request URL                | 备注                 |
| -------- | -------------------------- | -------------------- |
| GET      | /api/v1/terminal/commands/ | 查询资产会话命令记录 |

**(2)**   **Request Header**

| 键            | 值                                             | 备注                                                         |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Authorization | Token $TOKEN | 管理员的token信息。                                          |
| X-JMS-ORG     | 00000000-0000-0000-0000-000000000002           | 00000000-0000-0000-0000-000000000002为组织ID，此id号为默认组织：**Default**，留空则默认为 **Default** 组织。 |
| accept        | application/json,  text/plain, */*             | 告诉服务器接收的数据格式                                     |

**(3)**   **Query Parameters**

| 参数名    | 类型   | 描述           | 是否必选 | 默认值 |
| --------- | ------ | -------------- | -------- | ------ |
| user      | String | 用户名称       | 否       |        |
| asset     | String | 资产名称       | 否       |        |
| input     | String | 命令           | 否       |        |
| account   | String | 资产账号       | 否       |        |
| order     | String | 排序           | 否       |        |
| date_from | String | 开始时间       | 是       |        |
| date_to   | String | 结束时间       | 是       |        |
| offset    | String | 每一页显示条数 | 是       |        |
| limit     | String | 分页偏移量     | 是       |        |

**(4)**   **请求示例**

  curl --location --request GET 'https://example.jumpserver.org/api/v1/terminal/commands/?command_storage_id=edf09740-9c01-4df0-b449-76fce4a2a021&order=-timestamp&date_to=2026-05-14T15%3A59%3A59.000Z&date_from=2026-05-06T06%3A15%3A00.149Z&offset=0&limit=15&display=1&draw=1' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: application/json' \
--header 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'  

**(5)**   **返回示例**

  

```
{
    "count": 48,
    "next": "https://example.jumpserver.org/api/v1/terminal/commands/?command_storage_id=edf09740-9c01-4df0-b449-76fce4a2a021&date_from=2026-05-06T06%3A15%3A00.149Z&date_to=2026-05-14T15%3A59%3A59.000Z&display=1&draw=1&limit=15&offset=15&order=-timestamp",
    "previous": null,
    "results": [
        {
            "user": "opsuser(opsuser)",
            "asset": "Ops-Test-001(10.1.14.51)",
            "input": "jmsctl status",
            "session": "6abd9baa-97fd-46d5-b6be-359d1712ad03",
            "risk_level": {
                "value": 0,
                "label": "Accept"
            },
            "org_id": "00000000-0000-0000-0000-000000000002",
            "id": "36af0086-0bbc-438e-af33-443a573473d8",
            "account": "root(root)",
            "output": "NAME           IMAGE                                                        COMMAND                  SERVICE    CREATED        STATUS                  PORTS\njms_celery     registry.fit2cloud.com/jumpserver/core:v4.10.16-ee           \"./entrypoint.sh sta…\"   celery     22 hours ago   Up 22 hours (healthy)   8080/tcp\njms_chen       registry.fit2cloud.com/jumpserver/chen:v4.10.16-ee           \"./entrypoint.sh wisp\"   chen       22 hours ago   Up 22 hours (healthy)   8082/tcp\njms_core       registry.fit2cloud...tcp\njms_redis      redis:7.4.6-bookworm                                         \"docker-entrypoint.s…\"   redis      22 hours ago   Up 22 hours (healthy)   6379/tcp\njms_video      registry.fit2cloud.com/jumpserver/video-worker:v4.10.16-ee   \"./entrypoint.sh\"        video      22 hours ago   Up 22 hours (healthy)   9000/tcp\njms_web        registry.fit2cloud.com/jumpserver/web:v4.10.16-ee            \"/docker-entrypoint.…\"   web        22 hours ago   Up 22 hours (healthy)   0.0.0.0:80->80/tcp, [::]:80->80/tcp",
            "timestamp": 1778658476,
            "timestamp_display": "2026/05/13 15:47:56 +0800",
            "remote_addr": "10.1.10.35"
        }
     ]
}
```

**(6)**   **返回示例参数说明**

| 字段名称          | 数据类型 | 字段描述   | 备注 |
| ----------------- | -------- | ---------- | ---- |
| user              | String   | 用户名     |      |
| asset             | String   | 资产名     |      |
| input             | String   | 命令输入   |      |
| session           | String   | 会话id     |      |
| risk_level        | Object   | 命令等级   |      |
| account           | String   | 账号名     |      |
| output            | String   | 命令输出   |      |
| timestamp_display | String   | 执行时间   |      |
| timestamp         | Int      | 执行时间戳 |      |
| remote_addr       | String   | 远端地址   |      |



# **3.登录日志**



#### **3.1 查询登录日志**

**(1)**   **API** **地址**

| 请求方式 | Request URL                | 备注                 |
| -------- | -------------------------- | -------------------- |
| GET      | /api/v1/audits/login-logs/ | 查询用户登录日志记录 |

**(2)**   **Request Header**

| 键            | 值                                             | 备注                                                         |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Authorization | Token $TOKEN | 管理员的token信息。                                          |
| X-JMS-ORG     | 00000000-0000-0000-0000-000000000002           | 00000000-0000-0000-0000-000000000002为组织ID，此id号为默认组织：**Default**，留空则默认为 **Default** 组织。 |
| accept        | application/json,  text/plain, */*             | 告诉服务器接收的数据格式                                     |

**(3)**   **Query Parameters**

| 参数名    | 类型     | 描述           | 是否必选 | 默认值                    |
| --------- | -------- | -------------- | -------- | ------------------------- |
| username  | String   | 用户名         | 否       |                           |
| ip        | String   | 登录ip         | 否       |                           |
| city      | String   | 登录城市       | 否       |                           |
| type      | String   | 类型           | 否       | W：类型；T：终端；U：未知 |
| mfa       | Interger | MFA            | 否       | 0：禁用；1开启            |
| status    | Integer  | 状态           | 否       | 1：成功；2：失败          |
| date_from | String   | 开始时间       | 是       |                           |
| date_to   | String   | 结束时间       | 是       |                           |
| offset    | String   | 每一页显示条数 | 是       |                           |
| limit     | String   | 分页偏移量     | 是       |                           |

**(4)**   **请求示例**

​    

```
curl --location --request GET 'https://example.jumpserver.org/api/v1/audits/login-logs/?date_from=2026-05-06T08%3A00%3A33.329Z&date_to=2026-05-14T15%3A59%3A59.000Z&offset=0&limit=15&display=1&draw=1' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: application/json' \
--header 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'
```

**(5)**   **返回示例**

  

```
{
    "count": 4745,
    "next": "https://example.jumpserver.org/api/v1/audits/login-logs/?date_from=2026-05-06T08%3A00%3A33.329Z&date_to=2026-05-14T15%3A59%3A59.000Z&display=1&draw=1&limit=15&offset=15",
    "previous": null,
    "results": [
        {
            "id": "755beb43-0b22-4a42-8a87-6a9f1253a0d0",
            "username": "admin",
            "type": {
                "value": "T",
                "label": "Terminal"
            },
            "ip": "10.1.10.35",
            "city": "LAN",
            "user_agent": "Go-http-client/1.1",
            "mfa": {
                "value": 2,
                "label": "-"
            },
            "reason": "The username or password you entered is incorrect, please enter it again. You can also try 9999 times (The account will be tempo",
            "reason_display": "The username or password you entered is incorrect, please enter it again. You can also try 9999 times (The account will be tempo",
            "backend": "OpenID",
            "backend_display": "OpenID",
            "status": {
                "value": false,
                "label": "Failed"
            },
            "datetime": "2026/05/13 15:56:59 +0800"
        }
	]
}
```

**(6)**   **返回示例参数说明**

| 字段名称        | 数据类型          | 字段描述     | 备注 |
| --------------- | ----------------- | ------------ | ---- |
| count           | Int               | 总数         |      |
| next            | String            | 下一页链接   |      |
| previous        | String            | 上一页链接   |      |
| results         | List              | 数据         |      |
| id              | String            | id           |      |
| username        | String            | 用户名       |      |
| type            | Object            | 登录来源类型 |      |
| ip              | String            | 登录来源ip   |      |
| city            | Object            | 登录城市     |      |
| user_agent      | String            | 用户代理     |      |
| mfa             | Object            | mfa等级      |      |
| reason          | String            | 失败原因     |      |
| reason_display  | Int               | 失败报错     |      |
| status          | Object            | 登录状态     |      |
| datetime        | String(date-time) | 登录时间     |      |
| backend         | String            | 认证后端     |      |
| backend_display | String            | 认证后端说明 |      |

# 4.改密日志



#### **4.1 查询改密日志**

**(1)**   **API** **地址**

| 请求方式 | Request URL                          | 备注                 |
| -------- | ------------------------------------ | -------------------- |
| GET      | /api/v1/audits/password-change-logs/ | 查询用户改密日志记录 |

**(2)**   **Request Header**

| 键            | 值                                             | 备注                                                         |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Authorization | Token $TOKEN | 管理员的token信息。                                          |
| X-JMS-ORG     | 00000000-0000-0000-0000-000000000002           | 00000000-0000-0000-0000-000000000002为组织ID，此id号为默认组织：**Default**，留空则默认为 **Default** 组织。 |
| accept        | application/json,  text/plain, */*             | 告诉服务器接收的数据格式                                     |

**(3)**   **Query Parameters**

| 参数名      | 类型   | 描述           | 是否必选 | 默认值 |
| ----------- | ------ | -------------- | -------- | ------ |
| user        | String | 用户名         | 否       |        |
| change_by   | String | 修改者         | 否       |        |
| remote_addr | String | 远端地址       | 否       |        |
| date_from   | String | 开始时间       | 是       |        |
| date_to     | String | 结束时间       | 是       |        |
| offset      | String | 每一页显示条数 | 是       |        |
| limit       | String | 分页偏移量     | 是       |        |

**(4)**   **请求示例**

  

```
curl --location --request GET 'https://example.jumpserver.org/api/v1/audits/password-change-logs/?date_from=2026-05-06T08%3A00%3A33.329Z&date_from=2026-05-06T08%3A22%3A22.351Z&date_to=2026-05-14T15%3A59%3A59.000Z&offset=0&limit=15&display=1&draw=1' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: application/json' \
--header 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'
```

**(5)**   **返回示例**

  

```
{
    "count": 6,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "1cb669dd-05a4-4123-b366-403c97ce682a",
            "user": "opsuser(opsuser)",
            "change_by": "opsuser(opsuser)",
            "remote_addr": "10.1.10.35",
            "datetime": "2026/05/13 11:18:58 +0800"
        }
     ]
}
```

**(6)**   **返回示例参数说明**

| 参数名      | 类型   | 描述     | 是否必选 | 默认值 |
| ----------- | ------ | -------- | -------- | ------ |
| user        | String | 用户名   | 否       |        |
| change_by   | String | 修改者   | 否       |        |
| remote_addr | String | 远端地址 | 否       |        |
| datetime    | String | 修改时间 | 是       |        |

# **5.操作日志**



#### **5.1 查询操作日志记录**

**(1)**   **API** **地址**



| 请求方式 | Request URL                  | 备注                 |
| -------- | ---------------------------- | -------------------- |
| GET      | /api/v1/audits/operate-logs/ | 查询用户操作日志记录 |

**(2)**   **Request Header**

| 键            | 值                                             | 备注                                                         |
| ------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| Authorization | Token $TOKEN | 管理员的token信息。                                          |
| X-JMS-ORG     | 00000000-0000-0000-0000-000000000002           | 00000000-0000-0000-0000-000000000002为组织ID，此id号为默认组织：**Default**，留空则默认为 **Default** 组织。 |
| accept        | application/json,  text/plain, */*             | 告诉服务器接收的数据格式                                     |

**(3)**   **Query Parameters**



| 参数名        | 类型   | 描述           | 是否必选 | 默认值                                                       |
| ------------- | ------ | -------------- | -------- | ------------------------------------------------------------ |
| user          | String | 用户名         | 否       |                                                              |
| action        | String | 动作           | 否       | `view` - 查看 `update` - 更新 `delete` - 删除 `create` - 创建 `export` - 导出 `download` - 下载 `connect` - 连接 `login` - 登录 `change_password` - 改密 `accept` - 接受 `review` - 审批 `notice` - 通知 `reject` - 拒绝 `approve` - 同意 `close` - 关闭 `finished` - 结束 |
| resource_type | String | 资源类型       | 否       |                                                              |
| resource      | String | 资源           | 否       |                                                              |
| remote_addr   | String | 远端地址       | 否       |                                                              |
| date_from     | String | 开始时间       | 是       |                                                              |
| date_to       | String | 结束时间       | 是       |                                                              |
| offset        | String | 每一页显示条数 | 是       |                                                              |
| limit         | String | 分页偏移量     | 是       |                                                              |

**(4)**   **请求示例**

 

```
curl --location --request GET 'https://example.jumpserver.org/api/v1/audits/operate-logs/?date_from=2026-05-06T09%3A00%3A47.264Z&date_to=2026-05-14T15%3A59%3A59.000Z&offset=0&limit=15&display=1&draw=1' \
--header 'Authorization: Token $TOKEN' \
--header 'Accept: application/json' \
--header 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
--header 'User-Agent: Apifox/1.0.0 (https://apifox.com)' \
--header 'Host: example.jumpserver.org' \
--header 'Connection: keep-alive' \
--header 'Cookie: SESSION_COOKIE_NAME_PREFIX=$SESSION'
```

**(5)**   **返回示例**

  

```
{
    "count": 205,
    "next": "https://example.jumpserver.org/api/v1/audits/operate-logs/?date_from=2026-05-06T09%3A00%3A47.264Z&date_to=2026-05-14T15%3A59%3A59.000Z&display=1&draw=1&limit=15&offset=15",
    "previous": null,
    "results": [
        {
            "id": "a7499b33-f790-4bb5-9866-56accadeef60",
            "user": "opsuser(opsuser)",
            "action": {
                "value": "create",
                "label": "Create"
            },
            "resource_type": "User session",
            "resource": "opsuser(opsuser)(10.1.10.35)",
            "remote_addr": "10.1.10.35",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "org_name": "DEFAULT",
            "datetime": "2026/05/13 15:42:03 +0800"
        }
     ]
}
```

**(6)**   **返回示例参数说明**



| 字段名称     | 数据类型 | 字段描述   | 备注 |
| ------------ | -------- | ---------- | ---- |
| count        | Int      | 总数       |      |
| next         | String   | 下一页链接 |      |
| previous     | String   | 上一页链接 |      |
| results      | List     | 数据       |      |
| id           | String   | id         |      |
| user         | String   | 用户名     |      |
| action       | String   | 动作       |      |
| resouce_type | String   | 资源类型   |      |
| resouce      | String   | 资源       |      |
| remote_addr  | String   | 远端地址   |      |
| org_id       | String   | 组织id     |      |
| org_name     | String   | 组织名称   |      |
| datetime     | String   | 日期       |      |
