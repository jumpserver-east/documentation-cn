## /api/v1/health/

### GET
- **Description:** System health check interface, used to detect whether the service is alive and whether the basic dependencies are normal.

- **Request headers:**

| key | value | Remarks |
|----|----|------|
| Authorization | `Bearer <token>` | Required, authentication Token, the format is fixed to `Bearer <token>` |
| Accept | application/json | Optional |

- **Return parameters:**

| Field | Description | Remarks |
| --- | --- | --- |
| status | Type: Boolean, the overall availability status of the service | true=available |
| db_status | Type: Boolean, database available status | true=normal |
| db_time | Type: Float, database detection time (seconds) | Precision as a fraction of a second |
| redis_status | Type: Boolean, Redis available status | true=normal |
| redis_time | Type: Float, Redis detection time (seconds) | Precision as a fraction of a second |
| time | Type: Int, server current time (Unix timestamp, seconds) | Can be used for clock drift detection |


- **Response Example**
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

- **Request Example**

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
		print('health: No content returned (204/empty)')
		return
	r.raise_for_status()
	try:
		data = r.json()
		print('health information:')
		print(data)
	except ValueError:
		print('health: Not JSON content, original:')
		print(r.text)

if __name__ == '__main__':
	check_health()
```

- **Use Case:**

Scenario: The operations team connects JumpServer to the Prometheus/Zabbix monitoring platform, and the probe calls the health check interface every minute (set a 5-second timeout to prevent detection blocking). When the returned `db_status` or `redis_status` is false, an alarm is triggered.

```sh
curl -s --max-time 5 -X GET \
  -H 'Authorization: Bearer <token>' \
  -H 'X-JMS-ORG: <organizationID>' \
  -H 'Accept: application/json' \
  'https://demo.jumpserver.org/api/v1/health/'
```

> For complete integration scenarios, please refer to: [Practical Case: Health Check and Monitoring Alarm Integration](../examples/monitor_integration.md)

