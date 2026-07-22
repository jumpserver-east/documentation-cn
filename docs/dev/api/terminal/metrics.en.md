## /api/v1/terminal/components/metrics/

### GET
- **Description:**

Return terminal component metrics, including the count, online status, and active sessions for each component type, plus host lists grouped by severity.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Return parameters:** 

| Field | Description |
| --- | --- |
| total | Type: integer, the total number of components of this type |
| type | Type: string, component type identifier, such as `core`, `lion`, `koko`, `celery`, `video_worker`, etc. |
| session_active | Type: integer, current number of active sessions (session-related components may be 0) |
| high | Type: array[string], list of hostnames of instances with high risk/attention status (may be empty) |
| normal | Type: array[string], list of normal instance host names |
| offline | Type: array[string], list of offline instance host names |
| critical | Type: array[string], list of host names of severe exception instances |

- **Response example:** 

```json
[
	{
		"total": 1,
		"type": "celery",
		"session_active": 0,
		"high": [],
		"normal": ["[Celery]-mtls.example.internal"],
		"offline": [],
		"critical": []
	},
	{
		"total": 1,
		"type": "core",
		"session_active": 0,
		"high": [],
		"normal": ["[Core]-mtls.example.internal"],
		"offline": [],
		"critical": []
	}
]
```

- **Request Example**

**CURL**

```bash
curl -X GET \
	-H "Authorization: Bearer $TOKEN" \
	-H "X-JMS-ORG: $ORG" \
	https://example/api/v1/terminal/components/metrics/
```

**python**
```python
# Use AK/SK Access example (HTTPSignatureAuth)
import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL    = "https://example"          # Site root address
KEY_ID     = "your key id"              # Access Key ID
KEY_SECRET = "your key secret"          # Access Key Secret
ORG_ID     = "your org id"              # organization ID

def get_terminal_components_metrics():
	url = f"{API_URL}/api/v1/terminal/components/metrics/"
	gmt_form = "%a, %d %b %Y %H:%M:%S GMT"
	signature_headers = ['(request-target)', 'accept', 'date']
	headers = {
		"Content-Type": "application/json",
		"X-JMS-ORG": ORG_ID,
		"Date": datetime.utcnow().strftime(gmt_form)
	}
	auth = HTTPSignatureAuth(
		key_id=KEY_ID,
		secret=KEY_SECRET,
		algorithm="hmac-sha256",
		headers=signature_headers
	)
	try:
		resp = requests.get(url, auth=auth, headers=headers, timeout=10)
		resp.raise_for_status()
		data = resp.json()
		print(json.dumps(data, indent=2, ensure_ascii=False))
	except Exception as e:
		print(f"Request failed: {e}")

if __name__ == "__main__":
	get_terminal_components_metrics()
```

- **Use Case:**

Scenario: The operations monitoring platform regularly checks the health status of each terminal component of JumpServer every minute, extracts the component type and host name of offline or critical instances, and immediately pushes an alarm once access components such as koko and lion are found to be offline.

```sh
curl -s -X GET \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>' \
	https://example/api/v1/terminal/components/metrics/ \
	| jq '[.[] | select((.offline | length > 0) or (.critical | length > 0)) | {type, offline, critical}]'
```

> For complete integration scenarios, please refer to: [Practical Case: Health Check and Monitoring Alarm Integration](../examples/monitor_integration.md)


