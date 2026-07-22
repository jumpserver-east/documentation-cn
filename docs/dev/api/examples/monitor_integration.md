# 实战案例：健康检查与监控告警集成

## 场景说明

企业运维团队通常已建有统一监控平台（Zabbix / Prometheus / 夜莺等），希望将 JumpServer 也纳入统一监控体系：定期探测服务是否存活、数据库与 Redis 等基础依赖是否正常，并监控 koko、lion、celery 等各组件的在线状态与会话负载，一旦出现异常立即触发告警。本案例通过 JumpServer 提供的健康检查与组件指标接口，编写一个可被 crontab / Zabbix 等调度的探测脚本，用自动化探测替代人工巡检。

## 前置条件

- 已部署并可访问的 JumpServer 服务（本文以 `https://localhost` 为例）。
- 一个具有查看终端组件权限的账号（如系统管理员或系统审计员），并已创建其 API Key 或获取到有效的 Bearer Token，用于访问组件指标接口（健康检查接口无需认证）。
- 监控执行机可以访问 JumpServer 的 HTTPS 端口，且已安装 Python 3 与 `requests` 库（`pip install requests`）。
- 如需接入 Zabbix / 夜莺等平台，监控执行机上已部署对应 Agent，可执行自定义脚本。

## 涉及接口

| 请求方式 | 接口地址 | 用途 |
| --- | --- | --- |
| GET | `/api/v1/health/` | 服务健康检查：探测服务存活及数据库、Redis 等依赖状态 |
| GET | `/api/v1/terminal/components/metrics/` | 组件指标：各组件（core/koko/lion/celery 等）数量、在线/离线/异常主机列表与活跃会话数 |

> 各接口字段级明细可在 `https://<JumpServer地址>/api/docs` 在线文档中查看。

## 操作流程

### 第一步：探测服务健康状态

调用健康检查接口，探测 JumpServer 服务本身及数据库、Redis 是否正常。该接口无需认证即可访问，适合作为最基础的存活探测。

```sh
curl -X GET 'https://localhost/api/v1/health/'
```

正常时返回类似如下内容：

```json
{
    "status": true,
    "db_status": true,
    "db_time": 0.0032639503479003906,
    "redis_status": true,
    "redis_time": 0.0004906654357910156,
    "time": 1762484673
}
```

判定要点：

- `status` 为 `false`，或请求超时 / 非 200 响应，说明服务整体不可用；
- `db_status` / `redis_status` 为 `false`，说明对应依赖异常；
- `db_time` / `redis_time` 持续偏高（如超过 1 秒），说明依赖存在性能压力，可作为预警指标。

### 第二步：获取组件指标

调用组件指标接口，获取各组件（core、koko、lion、celery 等）的总数、在线 / 离线 / 异常主机列表及活跃会话数。该接口需要认证，请求头需携带 `Authorization` 与 `X-JMS-ORG`。

```sh
curl -X GET 'https://localhost/api/v1/terminal/components/metrics/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

返回示例（数组，每个元素对应一种组件类型）：

```json
[
    {
        "total": 1,
        "type": "koko",
        "session_active": 3,
        "high": [],
        "normal": ["[KoKo]-jms-node1.example.internal"],
        "offline": [],
        "critical": []
    },
    {
        "total": 1,
        "type": "celery",
        "session_active": 0,
        "high": [],
        "normal": ["[Celery]-jms-node1.example.internal"],
        "offline": [],
        "critical": []
    }
]
```

判定要点：

- `offline` / `critical` 列表非空，说明存在离线或严重异常的组件实例，应立即告警；
- `high` 列表非空，说明存在高负载 / 需关注的实例，可作为预警；
- `session_active` 可用于绘制会话负载趋势，辅助容量规划。

### 第三步：接入监控平台

将上述两步封装为一个探测脚本（见下节完整示例代码），约定退出码：`0` 表示一切正常，非 `0` 表示异常。这样即可直接被 crontab、Zabbix、夜莺等调度：

- **crontab**：定时执行脚本，配合退出码判断发送告警（邮件 / webhook 等）；
- **Zabbix**：将脚本配置为 Agent 自定义监控项（`UserParameter`）或外部检查脚本，对返回值 / 退出码做触发器告警；
- **Prometheus / 夜莺**：可在脚本基础上稍作改造，将指标写入 Pushgateway 或文本采集文件（textfile collector），由采集器统一抓取。

## 完整示例代码

以下 Python 脚本串联健康检查与组件指标两个接口，任一检查异常即以非 0 退出码结束，并将异常详情输出到 stderr，便于监控平台采集告警内容。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
JumpServer 健康检查与组件指标探测脚本

用法: python3 jms_monitor.py
退出码:
    0  一切正常
    1  服务健康检查异常 (服务不可用 / 数据库或 Redis 异常)
    2  组件指标异常 (存在离线或严重异常的组件实例)
    3  接口请求失败 (网络不通 / 认证失败等)
"""

import sys
import requests

# ==================== 配置区 ====================
API_URL = "https://localhost"                                  # JumpServer 地址
TOKEN   = "b96810faac725563304dada8c323c4fa061863d4"           # Bearer Token
ORG_ID  = "00000000-0000-0000-0000-000000000002"               # 组织 ID
TIMEOUT = 10                                                   # 请求超时 (秒)
VERIFY_TLS = False                                             # 自签名证书环境设为 False
# ================================================

AUTH_HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "X-JMS-ORG": ORG_ID,
    "Accept": "application/json",
}

errors = []      # 触发告警的异常项
warnings = []    # 仅提示、不触发退出码的预警项


def check_health():
    """检查服务健康状态: GET /api/v1/health/ (无需认证)"""
    url = f"{API_URL}/api/v1/health/"
    resp = requests.get(url, timeout=TIMEOUT, verify=VERIFY_TLS)
    resp.raise_for_status()
    data = resp.json()

    if not data.get("status"):
        errors.append("服务总体状态异常 (status=false)")
    if not data.get("db_status"):
        errors.append("数据库状态异常 (db_status=false)")
    if not data.get("redis_status"):
        errors.append("Redis 状态异常 (redis_status=false)")

    # 依赖响应耗时预警 (阈值可按需调整)
    for key, label in (("db_time", "数据库"), ("redis_time", "Redis")):
        cost = data.get(key)
        if isinstance(cost, (int, float)) and cost > 1:
            warnings.append(f"{label}探测耗时偏高: {cost:.3f} 秒")

    print(f"[health] status={data.get('status')} "
          f"db={data.get('db_status')} redis={data.get('redis_status')}")
    return len(errors) == 0


def check_components():
    """检查组件指标: GET /api/v1/terminal/components/metrics/ (需认证)"""
    url = f"{API_URL}/api/v1/terminal/components/metrics/"
    resp = requests.get(url, headers=AUTH_HEADERS,
                        timeout=TIMEOUT, verify=VERIFY_TLS)
    resp.raise_for_status()
    metrics = resp.json()

    ok = True
    for item in metrics:
        ctype = item.get("type", "unknown")
        offline = item.get("offline") or []
        critical = item.get("critical") or []
        high = item.get("high") or []

        if offline:
            errors.append(f"组件 {ctype} 存在离线实例: {', '.join(offline)}")
            ok = False
        if critical:
            errors.append(f"组件 {ctype} 存在严重异常实例: {', '.join(critical)}")
            ok = False
        if high:
            warnings.append(f"组件 {ctype} 存在高负载实例: {', '.join(high)}")

        print(f"[metrics] type={ctype} total={item.get('total')} "
              f"active_sessions={item.get('session_active')} "
              f"offline={len(offline)} critical={len(critical)}")
    return ok


def main():
    exit_code = 0
    try:
        if not check_health():
            exit_code = 1
    except requests.RequestException as e:
        print(f"健康检查接口请求失败: {e}", file=sys.stderr)
        sys.exit(3)

    try:
        if not check_components() and exit_code == 0:
            exit_code = 2
    except requests.RequestException as e:
        print(f"组件指标接口请求失败: {e}", file=sys.stderr)
        sys.exit(3)

    for w in warnings:
        print(f"WARNING: {w}")
    for e in errors:
        print(f"CRITICAL: {e}", file=sys.stderr)

    if exit_code == 0:
        print("JumpServer 检查通过, 一切正常")
    sys.exit(exit_code)


if __name__ == "__main__":
    # 关闭自签名证书告警输出 (生产环境建议配置可信证书并开启校验)
    if not VERIFY_TLS:
        requests.packages.urllib3.disable_warnings()
    main()
```

**crontab 部署示例**（每 5 分钟探测一次，异常时通过退出码触发告警脚本）：

```sh
# 编辑定时任务: crontab -e
*/5 * * * * /usr/bin/python3 /opt/scripts/jms_monitor.py >> /var/log/jms_monitor.log 2>&1 || /opt/scripts/send_alert.sh "JumpServer 监控异常, 详见 /var/log/jms_monitor.log"
```

**Zabbix Agent 接入示例**（自定义监控项，返回值非 0 时配置触发器告警）：

```sh
# /etc/zabbix/zabbix_agentd.d/jumpserver.conf
UserParameter=jumpserver.check,/usr/bin/python3 /opt/scripts/jms_monitor.py >/dev/null 2>&1; echo $?
```

## 常见问题

**Q1：组件指标接口返回 401，健康检查接口却能正常访问？**

A：`/api/v1/health/` 无需认证，而 `/api/v1/terminal/components/metrics/` 需要有效的认证信息。请检查 `Authorization: Bearer <token>` 中的 Token 是否有效、是否过期。若使用 API Key（AK/SK），需按 HTTP Signature 方式签名请求（可参考用户接口文档中的 `HTTPSignatureAuth` 完整示例）。

**Q2：组件指标接口返回 403 或数据为空？**

A：该接口需要具备查看终端组件的权限，普通用户通常无权访问。建议为监控专门创建一个具有系统审计员等只读角色的账号，避免直接使用超级管理员账号跑监控脚本。同时确认 `X-JMS-ORG` 传入的组织 ID 正确，组件指标为全局数据，通常使用默认组织即可。

**Q3：脚本在自签名证书环境报 SSL 证书校验错误？**

A：示例脚本提供了 `VERIFY_TLS` 开关，自签名环境可设为 `False`（对应 `requests` 的 `verify=False`）。生产环境建议为 JumpServer 配置可信证书并保持证书校验开启，避免中间人风险。

**Q4：`db_time` / `redis_time` 一直偏高，但 `status` 仍为 true，需要处理吗？**

A：需要关注。`status=true` 只代表当前可用，探测耗时持续偏高说明数据库或 Redis 存在性能压力，往往是故障前兆。建议把耗时作为预警指标（如示例脚本中超过 1 秒记 WARNING），并结合数据库慢查询、Redis 内存等指标进一步排查。
