# 实战案例：审计日志对接 SIEM 平台

## 场景说明

安全合规（等保、ISO 27001、内部审计等）通常要求堡垒机的审计数据长期留存并可关联分析，而 JumpServer 本身的日志保留周期有限。本案例通过 API 将 JumpServer 的四类审计数据——用户登录日志、操作日志、会话记录、命令记录——按时间增量持续同步到本地 JSON Lines 文件，再由 Filebeat/Logstash 等采集器送入 Splunk、ELK 或自建数仓，实现审计数据的长期留存与统一检索。同步脚本使用本地 checkpoint 文件记录上次同步位置，每次只拉取增量区间的数据，可通过 crontab 周期调度。

## 前置条件

- JumpServer V4 环境，网络上可从同步主机访问其 HTTPS 地址；
- 一个具备审计数据查看权限的账号（推荐系统审计员或管理员角色），并已创建 API Key（AK/SK）或获取 Bearer Token（参见 API 文档认证章节）；
- 明确要同步的组织：审计数据按组织隔离，请求头 `X-JMS-ORG` 决定拉取哪个组织的数据，多组织环境需要逐个组织执行；
- 同步主机安装 Python 3.7+，并安装依赖：`pip install requests httpsig`；
- （可选）已部署 Filebeat/Logstash 等采集器，用于把脚本输出的 JSON Lines 文件送入 SIEM。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/audits/login-logs/` | 获取用户登录日志 |
| GET | `/api/v1/audits/operate-logs/` | 获取操作日志 |
| GET | `/api/v1/terminal/sessions/` | 获取会话记录 |
| GET | `/api/v1/terminal/commands/` | 获取命令记录 |

四个接口均为标准分页接口（`limit` / `offset`，响应含 `count` / `next` / `previous` / `results`），但支持的时间过滤参数不同，增量拉取策略也因此不同：

| 接口 | 时间增量方式 |
| --- | --- |
| `/api/v1/terminal/commands/` | 服务端 `date_from` / `date_to`（ISO 8601 时间）精确过滤 |
| `/api/v1/audits/operate-logs/`、`/api/v1/terminal/sessions/` | 服务端 `days` 参数按天数粗过滤，客户端再按 checkpoint 精确截断 |
| `/api/v1/audits/login-logs/` | 无服务端时间过滤参数：用 `order=-datetime` 倒序翻页，客户端按 checkpoint 截断并提前终止翻页 |

> 各接口的字段级明细（查询参数、返回字段）可在 `https://<JumpServer地址>/api/docs` 在线文档查看。

## 操作流程

以下 curl 示例统一使用 Bearer Token 认证；所有请求都需要携带 `Authorization: Bearer <token>` 与 `X-JMS-ORG` 两个请求头。

### 第一步：拉取登录日志

登录日志接口没有时间过滤参数，因此用 `order=-datetime` 按登录时间倒序翻页：最新的数据排在最前，客户端只保留时间晚于 checkpoint 的记录，一旦某页记录全部早于 checkpoint 即停止翻页，避免全量深翻页。返回记录的关键字段有 `id`、`username`、`type`（登录方式）、`ip`、`city`、`mfa`、`status`、`datetime`（登录时间）等。

```sh
curl -X GET 'https://localhost/api/v1/audits/login-logs/?order=-datetime&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第二步：拉取操作日志

操作日志接口支持 `days` 参数（按最近天数粗过滤），把 checkpoint 距当前的时长向上取整为天数传入，先让服务端缩小数据量，再配合 `order=-datetime` 倒序与客户端精确截断完成增量。返回记录的关键字段有 `id`、`user`、`action`（动作）、`resource_type`、`resource`、`remote_addr`、`org_name`、`datetime` 等。

```sh
curl -X GET 'https://localhost/api/v1/audits/operate-logs/?days=1&order=-datetime&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第三步：拉取会话记录

会话接口同样支持 `days` 粗过滤，增量基准使用会话开始时间 `date_start`。注意会话有生命周期：`date_end` 在会话未结束时为空、`is_finished` 表示是否已结束，若 SIEM 端关心会话时长，可在入库后按 `id` 覆盖更新（本文示例按 `date_start` 增量导出一次）。关键字段有 `id`、`user`、`asset`、`account`、`protocol`、`login_from`、`remote_addr`、`is_finished`、`date_start`、`date_end` 等。

```sh
curl -X GET 'https://localhost/api/v1/terminal/sessions/?days=1&order=-date_start&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第四步：拉取命令记录

命令记录是四类数据中体量最大的，好在接口原生支持 `date_from` / `date_to`（ISO 8601 格式）服务端精确过滤，直接传入 `[checkpoint, now]` 区间即可，无需客户端截断。注意两点：`output`（命令输出）字段是 base64 编码，入库前可按需解码；`timestamp` 是 Unix 秒级时间戳，`timestamp_display` 才是可读时间。其余关键字段有 `id`、`user`、`asset`、`account`、`session`（所属会话 ID，可与第三步的会话数据关联）、`input`（命令内容）、`risk_level` 等。

```sh
curl -X GET 'https://localhost/api/v1/terminal/commands/?date_from=2026-07-21T00:00:00Z&date_to=2026-07-22T00:00:00Z&limit=100&offset=0' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

### 第五步：写入 JSON Lines 并推进 checkpoint

每类数据各自写入按天滚动的 `.jsonl` 文件（一行一条 JSON 记录，附加 `_log_type` 字段标识数据类型），供 Filebeat/Logstash 直接采集；每个数据源同步成功后才把自己的 checkpoint 推进到本轮窗口右边界（脚本启动时刻），某一源失败不影响其他源，下次运行自动补拉。输出示例（示意）：

```json
{"id": "1c92cc2b-6b90-4749-a9bb-e5372a1c26c9", "username": "admin", "type": {"value": "W", "label": "Web"}, "ip": "203.0.113.10", "datetime": "2026/07/22 10:48:46 +0800", "_log_type": "login_logs"}
```

## 完整示例代码

```python
# -*- coding: utf-8 -*-
# JumpServer 审计数据增量同步脚本：输出 JSON Lines，供 Filebeat/Logstash 采集入 SIEM
# 依赖：pip install requests httpsig
# 用法：python3 siem_export.py（建议配合 crontab 周期执行）

import json
import math
import os
import sys
from datetime import datetime, timedelta, timezone

import requests
from httpsig.requests_auth import HTTPSignatureAuth

# ======================= 基础配置 =======================
API_URL    = "https://localhost"                        # JumpServer 访问地址
KEY_ID     = "your id"                                  # API Key ID（AK）
KEY_SECRET = "your secret"                              # API Key Secret（SK）
ORG_ID     = "00000000-0000-0000-0000-000000000002"     # 组织 ID，多组织时逐个组织执行

PAGE_SIZE              = 100                            # 分页大小，不建议设置过大
DEFAULT_LOOKBACK_HOURS = 24                             # 首次运行（无 checkpoint）回溯时长
CHECKPOINT_FILE        = "/opt/jms-siem/checkpoint.json"
OUTPUT_DIR             = "/opt/jms-siem/output"
VERIFY_SSL             = False                          # 自签名证书环境置 False

if not VERIFY_SSL:
    requests.packages.urllib3.disable_warnings()

# ======================= 认证与请求 =======================
def build_auth():
    """AK/SK 签名认证；如用 Bearer Token，可去掉 auth，改在 headers 中添加 Authorization"""
    return HTTPSignatureAuth(
        key_id = KEY_ID, secret = KEY_SECRET,
        algorithm = "hmac-sha256",
        headers = ['(request-target)', 'accept', 'date']
    )

def build_headers():
    gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
    return {
        "Accept": "application/json",
        "X-JMS-ORG": ORG_ID,
        "Date": datetime.utcnow().strftime(gmt_form)
    }

def api_get(path, params):
    response = requests.get(
        f"{API_URL}{path}", auth = build_auth(), headers = build_headers(),
        params = params, verify = VERIFY_SSL, timeout = 60
    )
    response.raise_for_status()
    return response.json()

# ======================= 时间处理 =======================
def parse_dt(value):
    """兼容 JumpServer 常见时间格式：2026/07/22 10:48:46 +0800 与 ISO 8601"""
    if not value:
        return None
    v = str(value).strip().replace("Z", "+0000")
    for fmt in ("%Y/%m/%d %H:%M:%S %z",
                "%Y-%m-%dT%H:%M:%S.%f%z",
                "%Y-%m-%dT%H:%M:%S%z"):
        try:
            return datetime.strptime(v, fmt)
        except ValueError:
            continue
    return None

def to_utc_str(dt):
    return dt.astimezone(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")

# ======================= 分页拉取 =======================
def fetch_pages(path, params):
    """顺序翻完所有分页（配合服务端时间过滤使用）"""
    offset, results = 0, []
    while True:
        data = api_get(path, dict(params, limit = PAGE_SIZE, offset = offset))
        results.extend(data.get("results", []))
        if not data.get("next"):
            break
        offset += PAGE_SIZE
    return results

def fetch_desc_window(path, params, time_key, since, until):
    """
    倒序翻页，只保留 (since, until] 区间内的记录；
    当整页记录都早于 since 时提前终止，避免深翻页。
    """
    offset, results = 0, []
    while True:
        data = api_get(path, dict(params, limit = PAGE_SIZE, offset = offset))
        records = data.get("results", [])
        if not records:
            break
        all_older = True
        for r in records:
            dt = parse_dt(r.get(time_key))
            if dt is None:                  # 时间无法解析时保守保留，交给下游按 id 去重
                all_older = False
                results.append(r)
                continue
            if dt <= since:
                continue
            all_older = False
            if dt <= until:
                results.append(r)
        if all_older or not data.get("next"):
            break
        offset += PAGE_SIZE
    return results

def coarse_days(since, until):
    """把 checkpoint 距今时长换算为 days 参数（服务端粗过滤，客户端再精确截断）"""
    return max(1, math.ceil((until - since).total_seconds() / 86400))

# ======================= 四类数据源 =======================
def sync_login_logs(since, until):
    # 登录日志：接口无时间过滤参数，倒序翻页 + 客户端截断
    return fetch_desc_window("/api/v1/audits/login-logs/",
                             {"order": "-datetime"},
                             "datetime", since, until)

def sync_operate_logs(since, until):
    # 操作日志：days 粗过滤 + 倒序翻页 + 客户端截断
    return fetch_desc_window("/api/v1/audits/operate-logs/",
                             {"days": coarse_days(since, until), "order": "-datetime"},
                             "datetime", since, until)

def sync_sessions(since, until):
    # 会话记录：days 粗过滤，按会话开始时间 date_start 增量
    return fetch_desc_window("/api/v1/terminal/sessions/",
                             {"days": coarse_days(since, until), "order": "-date_start"},
                             "date_start", since, until)

def sync_commands(since, until):
    # 命令记录：date_from / date_to 服务端精确过滤
    return fetch_pages("/api/v1/terminal/commands/",
                       {"date_from": to_utc_str(since), "date_to": to_utc_str(until)})

# ======================= 输出与 checkpoint =======================
def write_jsonl(log_type, records):
    if not records:
        print(f"[{log_type}] 本次无新增数据")
        return
    os.makedirs(OUTPUT_DIR, exist_ok = True)
    day = datetime.now(timezone.utc).strftime("%Y%m%d")
    path = os.path.join(OUTPUT_DIR, f"jumpserver-{log_type}-{day}.jsonl")
    with open(path, "a", encoding = "utf-8") as f:
        for r in records:
            r["_log_type"] = log_type       # 便于 SIEM 端区分数据类型
            f.write(json.dumps(r, ensure_ascii = False) + "\n")
    print(f"[{log_type}] 新增 {len(records)} 条 -> {path}")

def load_checkpoints():
    if os.path.exists(CHECKPOINT_FILE):
        with open(CHECKPOINT_FILE, encoding = "utf-8") as f:
            return json.load(f)
    return {}

def save_checkpoints(cps):
    os.makedirs(os.path.dirname(CHECKPOINT_FILE), exist_ok = True)
    tmp = CHECKPOINT_FILE + ".tmp"
    with open(tmp, "w", encoding = "utf-8") as f:
        json.dump(cps, f, indent = 2)
    os.replace(tmp, CHECKPOINT_FILE)        # 原子替换，避免写一半损坏 checkpoint

# ======================= 主流程 =======================
def main():
    cps = load_checkpoints()
    until = datetime.now(timezone.utc)      # 本轮窗口右边界：脚本启动时刻
    default_since = until - timedelta(hours = DEFAULT_LOOKBACK_HOURS)

    jobs = [
        ("login_logs",   sync_login_logs),
        ("operate_logs", sync_operate_logs),
        ("sessions",     sync_sessions),
        ("commands",     sync_commands),
    ]
    failed = []
    for name, func in jobs:
        since = parse_dt(cps.get(name)) or default_since
        try:
            records = func(since, until)
            write_jsonl(name, records)
            cps[name] = to_utc_str(until)   # 该源成功后才推进它自己的 checkpoint
            save_checkpoints(cps)
        except Exception as e:
            failed.append(name)
            print(f"[{name}] 同步失败，checkpoint 未推进，下次运行自动补拉：{e}",
                  file = sys.stderr)

    if failed:
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### 部署：crontab 周期调度

```sh
# 每 5 分钟增量同步一次，输出追加到日志文件
*/5 * * * * /usr/bin/python3 /opt/jms-siem/siem_export.py >> /var/log/jms-siem-export.log 2>&1
```

### 部署：Filebeat 采集 JSON Lines（示例）

```yaml
filebeat.inputs:
  - type: filestream
    id: jumpserver-audit
    paths:
      - /opt/jms-siem/output/*.jsonl
    parsers:
      - ndjson:
          target: ""
          add_error_key: true
output.logstash:
  hosts: ["logstash.example.com:5044"]
```

## 常见问题

**Q1：数据量很大，一次同步要翻几百页怎么办？**

A：核心思路是让每个增量窗口尽量小，而不是加大单页容量。缩短 crontab 调度周期（如 5 分钟一次），`limit` 保持 100 左右即可；命令记录支持 `date_from` / `date_to`，可把大窗口再切成多个小时间段分段拉取；登录/操作日志依赖倒序翻页加提前终止，checkpoint 间隔越短翻页越浅。尽量避免大 `offset` 深翻页，数据库偏移越大查询越慢。

**Q2：时间格式和时区有什么坑？**

A：JumpServer 接口返回的时间可能是 `2026/07/22 10:48:46 +0800` 这类带时区偏移的格式，也可能是 ISO 8601，务必带时区解析（如脚本中的 `parse_dt`），内部统一转换为 UTC 再比较，切勿用无时区的本地时间直接对比，否则 checkpoint 会漏数据或重复。传给 `date_from` / `date_to` 的值使用带时区的 ISO 8601（如 `2026-07-22T00:00:00Z`）。命令记录的 `timestamp` 是 Unix 秒级时间戳，没有时区歧义，`timestamp_display` 才是格式化时间。

**Q3：checkpoint 文件丢了，重跑会不会造成重复数据？**

A：会重复拉取，但可以做到幂等：四类记录都有全局唯一的 `id`（UUID），SIEM 端以 `id` 作为唯一键即可自动去重（Elasticsearch 写入时把 `id` 设为文档 `_id`，Splunk 检索时用 `dedup id`）。checkpoint 丢失后脚本默认只回溯 `DEFAULT_LOOKBACK_HOURS`（24 小时），若需要补更早的数据，可临时调大该值重跑一次。建议把 checkpoint 文件纳入备份，并保留输出目录的 `.jsonl` 文件一段时间。

**Q4：请求返回 403，或拉到的数据不全？**

A：先检查账号权限——审计接口需要审计员或管理员级别的查看权限，普通用户只能看到自己的数据（如 `my-login-logs`）。再检查 `X-JMS-ORG`：审计数据按组织隔离，该请求头决定拉取哪个组织的数据，多组织环境必须遍历各组织 ID 分别执行同步，否则会漏掉其他组织的审计记录。
