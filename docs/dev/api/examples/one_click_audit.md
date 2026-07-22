# 实战案例：外部系统一键审计

## 场景说明

企业的安全运营 / 审计平台通常需要对堡垒机的操作行为做集中审计：给定"人员 + 时间段"（或"资产 + 时间段"），一键调取该维度下的全部审计数据——何时从哪里登录了系统、登录了哪些资产、在会话中执行了哪些命令，并汇总成一份审计报告。本案例通过编排 JumpServer 的登录日志、会话记录、命令记录等审计类 API，实现外部系统的"一键审计"能力，并给出可直接运行的 Python 脚本。

## 前置条件

- JumpServer V4 环境，外部系统与 JumpServer 网络可达；
- 一个具备审计权限的 API 账号：系统审计员 / 系统管理员角色（或具备相应审计查看权限的 RBAC 角色），普通用户只能查询到自己的记录；
- 认证凭据二选一：
    - API Key（AK/SK）：在 JumpServer「个人信息 - API Key」中创建，配合 `httpsig` 签名认证（推荐，长期有效）；
    - Bearer Token：请求头携带 `Authorization: Bearer <token>`；
- 审计数据按组织隔离，需要知道目标数据所在组织的 ID，通过请求头 `X-JMS-ORG` 指定（留空默认为 `Default` 组织）；跨组织审计需遍历组织 ID 分别调用；
- 完整示例需 Python 3.7+，并安装依赖：`pip install requests httpsig`；
- 若 JumpServer 使用自签名证书，`requests` 调用可按需设置 `verify=False`（生产环境建议配置可信证书）。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/users/users/` | 按用户名查询用户，获取 `user_id` |
| GET | `/api/v1/audits/login-logs/` | 查询登录日志（何时、从哪里、以何种方式登录） |
| GET | `/api/v1/terminal/sessions/` | 查询会话记录（登录了哪些资产、使用什么账号） |
| GET | `/api/v1/terminal/commands/` | 查询会话中的命令记录（执行了哪些命令） |
| GET | `/api/v1/terminal/sessions/{id}/replay/` | 获取会话录像文件地址 |
| GET | `/api/v1/terminal/sessions/{id}/replay/download/` | 下载会话录像文件 |

> 说明：以上接口的字段级明细（全部查询参数与返回字段）可在 JumpServer 在线接口文档 `https://<JumpServer地址>/api/docs` 中查看。

## 操作流程

所有请求需携带统一的认证请求头：

| 键 (Header) | 示例值 | 说明 |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | 认证 Token，格式固定为 `Bearer <token>`；使用 AK/SK 时改为 httpsig 签名 |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | 组织 ID，不传则默认归属 `Default` 组织 |

### 步骤一：按用户名定位用户，获取 user_id

会话记录接口按 `user_id`（UUID）过滤，而审计平台的输入通常是用户名，因此先通过用户列表接口的 `username` 参数把用户名解析成 `user_id`。

``` sh
curl -X GET 'https://localhost/api/v1/users/users/?username=zhangsan' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回列表中取第一个元素的 `id` 即为 `user_id`（如 `d461c2e0-95cd-4ccd-aee8-a5d767560eea`）。

### 步骤二：查询该用户的登录日志

登录日志接口记录用户对 JumpServer 平台本身的登录行为，支持按 `username`、`ip`、`city`、`type`（W=Web / T=Terminal / U=Unknown）、`status`（1=成功 / 0=失败）、`search` 等条件过滤。

| 参数名 | 类型 | 描述 | 是否必选 |
| --- | --- | --- | --- |
| username | String | 用户名 | 否 |
| ip | String | 登录来源 IP | 否 |
| status | Int | 登录状态，1 成功 / 0 失败 | 否 |
| type | String | 登录类型，W/T/U | 否 |
| search | String | 搜索关键字 | 否 |
| limit | Int | 每页条数 | 否 |
| offset | Int | 分页偏移量 | 否 |

``` sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?username=zhangsan&status=1&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回结果中每条记录的 `datetime` 字段为登录时间（如 `2026/07/21 10:17:20 +0800`），审计时间段的筛选在外部系统侧按该字段完成（见完整示例代码）。

### 步骤三：查询该用户的会话记录（登录了哪些资产）

会话记录接口是资产操作审计的主线索：一条会话对应"某用户使用某账号、以某协议登录某资产"的一次完整过程。支持的主要过滤参数：

| 参数名 | 类型 | 描述 | 是否必选 |
| --- | --- | --- | --- |
| user_id | String | 用户 ID，按人员审计时使用 | 否 |
| asset_id | String | 资产 ID，按资产审计时使用 | 否 |
| is_finished | Boolean | 是否已结束，true/false | 否 |
| protocol | String | 协议，如 ssh/rdp | 否 |
| login_from | String | 登录来源：ST/RT/WT/DT/VT | 否 |
| days | Number | 查询最近 N 天内的会话（服务端粗筛） | 否 |
| limit | Int | 每页条数 | 否 |
| offset | Int | 分页偏移量 | 否 |

按"人员 + 时间段"审计：

``` sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?user_id=d461c2e0-95cd-4ccd-aee8-a5d767560eea&days=30&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

按"资产 + 时间段"审计只需把 `user_id` 换成 `asset_id`：

``` sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?asset_id=4bdae07e-c214-4a12-a9db-be8146219bc8&days=30&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回的每条会话包含 `id`（会话 ID，下一步查命令用）、`asset`、`account`、`protocol`、`date_start`、`date_end`、`is_finished`、`has_command`（是否有命令记录）、`command_amount`（命令数量）、`has_replay`（是否有录像）等字段。精确的时间段过滤在外部系统侧按 `date_start` 字段完成，`days` 参数仅用于服务端粗筛以减少数据量。

### 步骤四：查询会话的命令明细

命令记录接口支持按 `session_id` 精确拉取单个会话内的全部命令，也支持 `date_from`/`date_to`（ISO 8601 格式，curl 中冒号需转义为 `%3A`）按时间段过滤：

| 参数名 | 类型 | 描述 | 是否必选 |
| --- | --- | --- | --- |
| session_id | String | 会话 ID | 否 |
| asset_id | String(UUID) | 资产 ID | 否 |
| user | String | 用户名 | 否 |
| account | String | 资产账号 | 否 |
| input | String | 按命令内容检索 | 否 |
| risk_level | Int | 风险等级：0 接受 / 4 警告 / 5 拒绝 / 6-8 复核相关 | 否 |
| date_from | String(date-time) | 开始时间 | 否 |
| date_to | String(date-time) | 结束时间 | 否 |
| limit | Int | 每页条数 | 否 |
| offset | Int | 分页偏移量 | 否 |

``` sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?session_id=8608a7af-f1ee-4f84-bb8a-1384b71914f5&date_from=2026-07-01T00%3A00%3A00.000Z&date_to=2026-07-21T23%3A59%3A59.999Z&offset=0&limit=100' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回的每条命令包含 `input`（命令输入）、`output`（命令输出）、`risk_level`（风险等级）、`timestamp_display`（执行时间）、`remote_addr`（来源地址）等字段，可直接写入审计报告。

### 步骤五（可选）：获取会话录像

对 `has_replay` 为 true 的会话，可通过 API 获取录像文件地址或直接下载录像文件，作为审计报告的附件留存：

``` sh
# 获取录像文件地址（返回 JSON，file 字段为录像文件下载地址）
curl -X GET 'https://localhost/api/v1/terminal/sessions/8608a7af-f1ee-4f84-bb8a-1384b71914f5/replay/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'

# 直接下载录像文件
curl -X GET 'https://localhost/api/v1/terminal/sessions/8608a7af-f1ee-4f84-bb8a-1384b71914f5/replay/download/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
    -o replay_8608a7af.tar
```

如需在线回放，请登录 JumpServer Web 控制台，在「审计台 - 会话审计」页面找到对应会话，点击"回放"即可在浏览器中播放。

## 完整示例代码

以下脚本输入"用户名 + 审计时间段"，自动完成：解析用户 → 拉取登录日志 → 拉取会话清单 → 逐会话拉取命令明细，最终生成一份 JSON 审计报告文件。

```python
# -*- coding: utf-8 -*-
"""
JumpServer 一键审计脚本
按 "用户名 + 时间段" 拉取审计数据:
  1. 按用户名解析 user_id
  2. 拉取该用户的登录日志(本地按 datetime 过滤时间段)
  3. 拉取该用户的会话记录(days 服务端粗筛 + 本地按 date_start 精筛)
  4. 逐会话拉取命令明细(服务端 date_from/date_to 过滤)
  5. 汇总输出 JSON 审计报告

依赖: Python 3.7+, pip install requests httpsig
"""

import json
import sys
from datetime import datetime

import requests
from httpsig.requests_auth import HTTPSignatureAuth

# ======== 基本配置(按实际环境修改) ========
API_URL    = "https://localhost"
KEY_ID     = "your id"       # API Key ID(个人信息 - API Key 中创建)
KEY_SECRET = "your secret"   # API Key Secret
ORG_ID     = "00000000-0000-0000-0000-000000000002"  # 组织 ID, 默认组织

# ======== 审计输入(人员 + 时间段) ========
USERNAME  = "zhangsan"
DATE_FROM = "2026-07-01T00:00:00.000Z"   # 审计开始时间(ISO 8601)
DATE_TO   = "2026-07-21T23:59:59.999Z"   # 审计结束时间(ISO 8601)

PAGE_SIZE = 100


def build_auth():
    """AK/SK 签名认证; 如使用 Bearer Token, 可去掉 auth,
    在请求头中增加 Authorization: Bearer <token> 即可"""
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = ['(request-target)', 'accept', 'date'],
    )


def build_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Accept": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form),
    }


def api_get_all(path, params=None):
    """按 limit/offset 循环翻页, 拉取列表接口的全部数据"""
    params = dict(params or {})
    params["limit"] = PAGE_SIZE
    params["offset"] = 0
    results = []
    while True:
        try:
            resp = requests.get(
                f"{API_URL}{path}",
                auth = build_auth(),
                headers = build_headers(),
                params = params,
            )
            resp.raise_for_status()
        except requests.RequestException as e:
            print(f"API 请求失败: {path}, 错误: {e}")
            sys.exit(1)
        data = resp.json()
        results.extend(data.get("results", []))
        if not data.get("next"):     # next 为 null 表示已到最后一页
            return results
        params["offset"] += PAGE_SIZE


def parse_dt(value):
    """兼容 JumpServer 返回的时间格式(2026/07/21 10:17:20 +0800)与 ISO 8601"""
    for fmt in ("%Y/%m/%d %H:%M:%S %z",
                "%Y-%m-%dT%H:%M:%S.%f%z",
                "%Y-%m-%dT%H:%M:%S%z"):
        try:
            return datetime.strptime(value, fmt)
        except (TypeError, ValueError):
            continue
    return None


def within(value, start, end):
    """判断时间字符串是否落在审计时间段内"""
    dt = parse_dt(value)
    return dt is not None and start <= dt <= end


def main():
    start = parse_dt(DATE_FROM)
    end = parse_dt(DATE_TO)
    if not start or not end:
        print("DATE_FROM / DATE_TO 格式错误, 应为 ISO 8601, 如 2026-07-01T00:00:00.000Z")
        sys.exit(1)

    # 1. 按用户名解析 user_id
    users = api_get_all("/api/v1/users/users/", {"username": USERNAME})
    if not users:
        print(f"未找到用户: {USERNAME}")
        sys.exit(1)
    user = users[0]
    user_id = user["id"]
    print(f"[1/4] 定位用户: {user.get('name')}({user.get('username')}), id={user_id}")

    # 2. 登录日志: 服务端按 username 过滤, 时间段在本地按 datetime 字段过滤
    login_logs = api_get_all("/api/v1/audits/login-logs/", {"username": USERNAME})
    login_logs = [x for x in login_logs if within(x.get("datetime"), start, end)]
    print(f"[2/4] 时间段内登录日志 {len(login_logs)} 条")

    # 3. 会话记录: 服务端按 user_id + days 粗筛, 本地按 date_start 精筛
    days = max((datetime.now(start.tzinfo) - start).days + 1, 1)
    sessions = api_get_all("/api/v1/terminal/sessions/",
                           {"user_id": user_id, "days": days})
    sessions = [x for x in sessions if within(x.get("date_start"), start, end)]
    print(f"[3/4] 时间段内会话 {len(sessions)} 个")

    # 4. 逐会话拉取命令明细(服务端支持 session_id + date_from/date_to 过滤)
    session_reports = []
    for s in sessions:
        commands = []
        if s.get("has_command"):     # 图形/SFTP 等会话没有命令记录, 跳过
            commands = api_get_all("/api/v1/terminal/commands/", {
                "session_id": s["id"],
                "date_from": DATE_FROM,
                "date_to": DATE_TO,
            })
        session_reports.append({
            "session_id": s["id"],
            "asset": s.get("asset"),
            "account": s.get("account"),
            "protocol": s.get("protocol"),
            "remote_addr": s.get("remote_addr"),
            "date_start": s.get("date_start"),
            "date_end": s.get("date_end"),
            "is_finished": s.get("is_finished"),
            "has_replay": s.get("has_replay"),
            # 有录像的会话, 可用该地址下载录像文件作为审计附件
            "replay_download": (
                f"{API_URL}/api/v1/terminal/sessions/{s['id']}/replay/download/"
                if s.get("has_replay") else None
            ),
            "command_count": len(commands),
            "commands": [
                {
                    "input": c.get("input"),
                    "output": (c.get("output") or "")[:200],  # 截断输出, 避免报告过大
                    "risk_level": (c.get("risk_level") or {}).get("label"),
                    "timestamp_display": c.get("timestamp_display"),
                    "remote_addr": c.get("remote_addr"),
                }
                for c in commands
            ],
        })
    print("[4/4] 命令明细拉取完成")

    # 5. 汇总审计报告
    report = {
        "audit_target": {"username": USERNAME, "user_id": user_id},
        "audit_range": {"date_from": DATE_FROM, "date_to": DATE_TO},
        "generated_at": datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%SZ"),
        "summary": {
            "login_count": len(login_logs),
            "session_count": len(session_reports),
            "command_count": sum(x["command_count"] for x in session_reports),
        },
        "login_logs": [
            {
                "datetime": x.get("datetime"),
                "ip": x.get("ip"),
                "city": x.get("city"),
                "type": (x.get("type") or {}).get("label"),
                "status": (x.get("status") or {}).get("label"),
                "backend": x.get("backend_display"),
            }
            for x in login_logs
        ],
        "sessions": session_reports,
    }

    out_file = f"audit_report_{USERNAME}.json"
    with open(out_file, "w", encoding="utf-8") as f:
        json.dump(report, f, ensure_ascii = False, indent = 2)
    print(f"审计报告已生成: {out_file}")
    print("汇总: " + json.dumps(report["summary"], ensure_ascii = False))


if __name__ == "__main__":
    main()
```

按"资产 + 时间段"审计时，只需把步骤 3 中的过滤参数 `{"user_id": user_id}` 替换为 `{"asset_id": "<资产ID>"}`（资产 ID 可通过资产列表接口按名称/IP 检索获得），其余流程完全一致。

## 常见问题

**Q1：调用审计接口返回 403，或查不到应有的数据？**

A：审计类接口需要系统审计员 / 系统管理员角色（或具备审计查看权限的 RBAC 角色），普通用户身份只能查询到自己的记录。另外审计数据按组织隔离，请求头 `X-JMS-ORG` 必须指向数据所在组织的 ID；用户在多个组织有操作记录时，需遍历组织 ID 分别调用后合并。

**Q2：会话记录、登录日志如何按时间段过滤？**

A：接口定义中，`/api/v1/terminal/commands/` 支持 `date_from`/`date_to` 服务端过滤（ISO 8601 格式，curl 中冒号需转义为 `%3A`）；`/api/v1/terminal/sessions/` 未定义这两个参数，可先用 `days` 参数按"最近 N 天"粗筛，再在外部系统侧按返回的 `date_start` 字段精筛；`/api/v1/audits/login-logs/` 按 `username` 等条件过滤后，在外部系统侧按 `datetime` 字段筛选时间段。完整示例代码即按此方式处理。

**Q3：为什么有的会话查不到命令、下载不到录像？**

A：只有命令行类协议（如 SSH、数据库命令行）的会话才产生命令记录，SFTP、RDP 等文件/图形类会话 `has_command` 为 false、命令数为 0，属正常现象；录像仅在会话 `has_replay`/`can_replay` 为 true 时存在，未结束（`is_finished=false`）或组件不支持录像的会话无法下载。在线回放可在 Web 控制台「审计台 - 会话审计」页面点击对应会话的"回放"。

**Q4：列表接口一次最多能返回多少条数据？**

A：所有列表接口统一分页，返回 `count/next/previous/results` 结构，通过 `limit`（每页条数）与 `offset`（偏移量）控制翻页，`next` 为 null 表示已到最后一页。一键审计场景务必像示例脚本一样循环翻页拉取全量数据，只取第一页会造成审计数据缺漏。
