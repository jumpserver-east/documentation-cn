# Practical case: health check and monitoring alarm integration

## Scene description

Enterprise operations teams usually have built a unified monitoring platform (Zabbix / Prometheus / Nightingale, etc.) and hope to incorporate JumpServer into the unified monitoring system: regularly detect whether the service is alive and whether basic dependencies such as database and Redis are normal, and monitor the online status and session load of components such as koko, lion, celery, etc., and trigger alarms immediately if an abnormality occurs. This case uses the health check and component indicator interface provided by JumpServer to write a detection script that can be scheduled by crontab / Zabbix, etc., and replace manual inspection with automated detection.

## Preconditions

- The JumpServer service has been deployed and accessible (this article uses `https://localhost` as an example).
- An account (such as a system administrator or system auditor) with permission to view terminal components, and has created its API Key or obtained a valid Bearer Token, which is used to access the component indicator interface (the health check interface does not require authentication).
- The monitoring execution machine can access the HTTPS port of JumpServer, and has installed Python 3 and the `requests` library (`pip install requests`).
- If you need to access platforms such as Zabbix/Nightingale, the corresponding Agent has been deployed on the monitoring execution machine and you can execute custom scripts.

## Involving interface

| Request method | interface address | Purpose |
| --- | --- | --- |
| GET | `/api/v1/health/` | Service health check: detect service survival and dependency status of database, Redis, etc. |
| GET | `/api/v1/terminal/components/metrics/` | Component indicators: number of each component (core/koko/lion/celery, etc.), online/offline/abnormal host list and number of active sessions |

> Field-level details for each endpoint are available in the online documentation at `https://<JumpServer-address>/api/docs`.

## Operation process

### Step 1: Detect service health status

Call the health check interface to detect whether the JumpServer service itself, database, and Redis are normal. This interface can be accessed without authentication and is suitable as the most basic survival detection.

```sh
curl -X GET 'https://localhost/api/v1/health/'
```

Normally, content similar to the following is returned:

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

Judgment points:

- `status` is `false`, or the request times out/non-200 response, indicating that the service as a whole is unavailable;
- `db_status` / `redis_status` is `false`, indicating a corresponding dependency exception;
- If `db_time` / `redis_time` continues to be high (for example, more than 1 second), it indicates that there is performance pressure on the dependency and can be used as an early warning indicator.

### Step 2: Get component metrics

Call the component indicator interface to obtain the total number of each component (core, koko, lion, celery, etc.), the list of online/offline/abnormal hosts, and the number of active sessions. This interface requires authentication, and the request header must carry `Authorization` and `X-JMS-ORG`.

```sh
curl -X GET 'https://localhost/api/v1/terminal/components/metrics/' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

Return example (array, each element corresponds to a component type):

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

Judgment points:

- The `offline` / `critical` list is not empty, indicating that there are offline or seriously abnormal component instances, and an alarm should be issued immediately;
- The `high` list is not empty, indicating that there are high load/instances requiring attention, which can be used as an early warning;
- `session_active` can be used to draw session load trends and assist capacity planning.

### Step 3: Access the monitoring platform

Encapsulate the above two steps into a detection script (see the complete sample code in the next section), and agree on the exit code: `0` means everything is normal, non-`0` means abnormality. In this way, it can be directly scheduled by crontab, Zabbix, Nightingale, etc.:

- **crontab**: Execute the script regularly, and send alerts (email/webhook, etc.) based on the exit code judgment;
- **Zabbix**: Configure the script as an Agent custom monitoring item (`UserParameter`) or an external inspection script, and trigger alarms for return values/exit codes;
- **Prometheus/Nightingale**: The script can be slightly modified to write indicators into Pushgateway or textfile collector, and the collector will uniformly capture them.

## Complete sample code

The following Python script connects the health check and component indicator interfaces in series. Any check exception will end with a non-0 exit code, and the exception details will be output to stderr to facilitate the monitoring platform to collect alarm content.

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
JumpServer Health check and component indicator detection script

Usage: python3 jms_monitor.py
exit code:
    0  everything is fine
    1  Service health check exception (Service unavailable / database or Redis Abnormal)
    2  Abnormal component indicators (Component instances that are offline or severely abnormal)
    3  Interface request failed (The network is blocked / Authentication failure, etc.)
"""

import sys
import requests

# ==================== Configuration area ====================
API_URL = "https://localhost"                                  # JumpServer Address
TOKEN   = "b96810faac725563304dada8c323c4fa061863d4"           # Bearer Token
ORG_ID  = "00000000-0000-0000-0000-000000000002"               # organization ID
TIMEOUT = 10                                                   # Request timeout (seconds)
VERIFY_TLS = False                                             # The self-signed certificate environment is set to False
# ================================================

AUTH_HEADERS = {
    "Authorization": f"Bearer {TOKEN}",
    "X-JMS-ORG": ORG_ID,
    "Accept": "application/json",
}

errors = []      # Exception items that trigger alarms
warnings = []    # Prompt only、Warning items that do not trigger exit codes


def check_health():
    """Check service health status: GET /api/v1/health/ (No authentication required)"""
    url = f"{API_URL}/api/v1/health/"
    resp = requests.get(url, timeout=TIMEOUT, verify=VERIFY_TLS)
    resp.raise_for_status()
    data = resp.json()

    if not data.get("status"):
        errors.append("The overall status of the service is abnormal (status=false)")
    if not data.get("db_status"):
        errors.append("Abnormal database status (db_status=false)")
    if not data.get("redis_status"):
        errors.append("Redis Abnormal status (redis_status=false)")

    # Dependency response time-consuming warning (Thresholds can be adjusted as needed)
    for key, label in (("db_time", "database"), ("redis_time", "Redis")):
        cost = data.get(key)
        if isinstance(cost, (int, float)) and cost > 1:
            warnings.append(f"{label}Detection time is high: {cost:.3f} seconds")

    print(f"[health] status={data.get('status')} "
          f"db={data.get('db_status')} redis={data.get('redis_status')}")
    return len(errors) == 0


def check_components():
    """Check component metrics: GET /api/v1/terminal/components/metrics/ (Authentication required)"""
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
            errors.append(f"components {ctype} There is an offline instance: {', '.join(offline)}")
            ok = False
        if critical:
            errors.append(f"components {ctype} There are serious abnormal instances: {', '.join(critical)}")
            ok = False
        if high:
            warnings.append(f"components {ctype} There is a high load instance: {', '.join(high)}")

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
        print(f"Health check interface request failed: {e}", file=sys.stderr)
        sys.exit(3)

    try:
        if not check_components() and exit_code == 0:
            exit_code = 2
    except requests.RequestException as e:
        print(f"Component indicator interface request failed: {e}", file=sys.stderr)
        sys.exit(3)

    for w in warnings:
        print(f"WARNING: {w}")
    for e in errors:
        print(f"CRITICAL: {e}", file=sys.stderr)

    if exit_code == 0:
        print("JumpServer Check passed, everything is fine")
    sys.exit(exit_code)


if __name__ == "__main__":
    # Turn off self-signed certificate alarm output (For production environments, it is recommended to configure a trusted certificate and enable verification.)
    if not VERIFY_TLS:
        requests.packages.urllib3.disable_warnings()
    main()
```

**crontab deployment example** (detected every 5 minutes, triggering an alarm script through the exit code when abnormal):

```sh
# Edit scheduled tasks: crontab -e
*/5 * * * * /usr/bin/python3 /opt/scripts/jms_monitor.py >> /var/log/jms_monitor.log 2>&1 || /opt/scripts/send_alert.sh "JumpServer Monitoring anomalies, See details /var/log/jms_monitor.log"
```

**Zabbix Agent access example** (customized monitoring items, configure trigger alarms when the return value is non-0):

```sh
# /etc/zabbix/zabbix_agentd.d/jumpserver.conf
UserParameter=jumpserver.check,/usr/bin/python3 /opt/scripts/jms_monitor.py >/dev/null 2>&1; echo $?
```

## FAQ

**Q1: The component indicator interface returns 401, but the health check interface can be accessed normally? **

A: `/api/v1/health/` does not require authentication, while `/api/v1/terminal/components/metrics/` requires valid authentication information. Please check whether the Token in `Authorization: Bearer <token>` is valid and expired. If you use API Key (AK/SK), you need to sign the request in HTTP Signature mode (refer to the `HTTPSignatureAuth` complete example in the user interface document).

**Q2: Does the component indicator interface return 403 or the data is empty? **

A: This interface requires permission to view terminal components, and ordinary users usually do not have permission to access it. It is recommended to create an account with read-only roles such as system auditor for monitoring, and avoid directly using the super administrator account to run monitoring scripts. At the same time, confirm that the organization ID passed in `X-JMS-ORG` is correct. The component indicators are global data. Usually, the default organization can be used.

**Q3: Does the script report an SSL certificate verification error in a self-signed certificate environment? **

A: The sample script provides the `VERIFY_TLS` switch, and the self-signed environment can be set to `False` (corresponding to `verify=False` of `requests`). For production environments, it is recommended to configure a trusted certificate for JumpServer and keep certificate verification turned on to avoid man-in-the-middle risks.

**Q4: `db_time` / `redis_time` has always been high, but `status` is still true. Does it need to be dealt with? **

A: Need attention. `status=true` only means that it is currently available. Continuously high detection time indicates that there is performance pressure on the database or Redis, which is often a precursor to failure. It is recommended to use time-consuming as an early warning indicator (for example, if it exceeds 1 second in the sample script, a WARNING is recorded), and further investigate based on indicators such as database slow query and Redis memory.
