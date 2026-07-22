## /api/v1/health/

### GET
- **描述：** 系统健康检查接口，用于探测服务是否存活及基础依赖是否正常。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
|----|----|------|
| Authorization | `Bearer <token>` | 必选，认证 Token，格式固定为 `Bearer <token>` |
| Accept | application/json | 可选 |

- **返回参数：**

| 字段 | 描述 | 备注 |
| --- | --- | --- |
| status | 类型：Boolean，服务总体可用状态 | true=可用 |
| db_status | 类型：Boolean，数据库可用状态 | true=正常 |
| db_time | 类型：Float，数据库探测耗时 (秒) | 精度为秒的小数 |
| redis_status | 类型：Boolean，Redis 可用状态 | true=正常 |
| redis_time | 类型：Float，Redis 探测耗时 (秒) | 精度为秒的小数 |
| time | 类型：Int，服务器当前时间 (Unix 时间戳, 秒) | 可用于时钟漂移检测 |


- **响应示例**
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

- **请求示例**

**CURL**
```sh
curl -X GET -H "Authorization: Bearer $TOKEN" 'https://demo.jumpserver.org/api/v1/health/'
```

**Python**
```python
import requests

API_URL = 'https://demo.jumpserver.org'
TOKEN = 'your token'

def check_health():
	url = f"{API_URL}/api/v1/health/"
	headers = {"Authorization": f"Bearer {TOKEN}"}
	r = requests.get(url, headers=headers, timeout=5)
	if r.status_code == 204 or not r.content:
		print('健康: 无内容返回 (204/空)')
		return
	r.raise_for_status()
	try:
		data = r.json()
		print('健康信息:')
		print(data)
	except ValueError:
		print('健康: 非 JSON 内容, 原始:')
		print(r.text)

if __name__ == '__main__':
	check_health()
```

- **使用案例：**

场景：运维团队将 JumpServer 接入 Prometheus/Zabbix 监控平台，探针每分钟调用一次健康检查接口（设置 5 秒超时防止探测阻塞），当返回中 `db_status` 或 `redis_status` 为 false 时触发告警。

```sh
curl -s --max-time 5 -X GET \
  -H 'Authorization: Bearer <token>' \
  -H 'X-JMS-ORG: <组织ID>' \
  -H 'Accept: application/json' \
  'https://demo.jumpserver.org/api/v1/health/'
```

> 完整集成场景可参考：[实战案例：健康检查与监控告警集成](../examples/monitor_integration.md)

