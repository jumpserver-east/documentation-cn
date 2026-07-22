## /api/v1/audits/job-logs/

### GET

- **Description:**
Query job logs corresponding to the **Job Audit** page under **Audit**. They record the content, executor, start and end times, and results of commands (Adhoc), Playbooks, file uploads, and other tasks run by Job Center.

- **Request headers:**

| Key (Header) | Example value | Description |
| ----------- | ------ | ---- |
| Authorization | `Bearer b96810faac725563304dada8c323c4fa061863d4` | Authentication Token, the example is administrator token; the format is fixed as `Bearer <token>` |
| X-JMS-ORG | `00000000-0000-0000-0000-000000000002` | Organization ID. If not passed, it will default to the `Default` organization. |
| Content-Type | `application/json` | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Optional value |
| --- | --- | --- |
| creator__name | Type: String, filter by creator (executor) name | - |
| material | Type: String, filtered by execution content (command/script) | - |
| search | Type: String, search term (fuzzy match) | - |
| order | Type: string, sort field | Such as `date_start` (ascending order) / `-date_start` (descending order) |
| limit | Type: int, number of items displayed on each page | - |
| offset | Type: int, paging offset | - |

- **Return parameters:**

| Field name | Description | Remarks |
| --- | --- | --- |
| count | Type: int, total | Total number of paging records |
| next | Type: string, next page link | No more pages is null |
| previous | Type: string, previous page link | No previous page is null |
| results | Type: list, job log list | The list elements are job log objects (see below) |
| id | Type: string, log ID | UUID |
| material | Type: string, execution content (command/script) | Can be null |
| job_type | Type: string, job type | adhoc (command)/playbook (script)/upload_file (file upload), default adhoc |
| time_cost | Type: int, execution time | Unit: seconds |
| is_finished | Type: boolean, whether it has been completed |    |
| is_success | Type: boolean, whether the execution is successful or not |    |
| task_id | Type: string, task ID | UUID, may be null |
| creator_name | Type: string, creator (executor) |    |
| org_id | Type: string, organization ID |    |
| org_name | Type: string, organization name |    |
| date_start | Type: string(date-time), start time | Can be null |
| date_finished | Type: string(date-time), end time | Can be null |
| date_created | Type: string(date-time), creation time |    |

- **Response example:**

> The examples are schematic data, and the fields are subject to online /api/docs.

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "3f7b9c2e-8d41-4a5b-9c6d-1e2f3a4b5c6d",
            "material": "df -h",
            "job_type": "adhoc",
            "time_cost": 6,
            "is_finished": true,
            "is_success": true,
            "task_id": "9a8b7c6d-5e4f-4a3b-8c1d-0e9f8a7b6c5d",
            "creator_name": "zhangsan",
            "org_id": "00000000-0000-0000-0000-000000000002",
            "org_name": "Default",
            "date_start": "2026-07-21 22:05:10 +0800",
            "date_finished": "2026-07-21 22:05:16 +0800",
            "date_created": "2026-07-21 22:05:09 +0800"
        }
    ]
}
```

- **Request Example**

**CURL**

``` sh
curl -X GET 'https://localhost/api/v1/audits/job-logs/?offset=0&limit=15&order=-date_start' \
    -H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

**Python**

```python
# Python Example

import requests
import json
from datetime import datetime
from httpsig.requests_auth import HTTPSignatureAuth

API_URL     = "https://localhost"
KEY_ID      = "your id"
KEY_SECRET  = "your secret"
ORG_ID      = "your org id"

def get_job_logs():
    url = f"{API_URL}/api/v1/audits/job-logs/"
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
    params = {
        "offset": 0,
        "limit": 15,
        "order": "-date_start"
    }

    try:
        response = requests.get(
            url, auth = auth, headers = headers, params = params
        )
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        print(f"API Request failed:{e}")
        return None

if __name__ == "__main__":
    result = get_job_logs()
    print(json.dumps(result, indent = 2, ensure_ascii = False))
```

- **Use Case:**

Scenario: After the change window ends, the operations personnel on duty pull the job logs executed in batches by zhangsan that night in reverse order of the start time, intercept the data of the change window period based on `date_start` on the client, and check the `is_finished` and `is_success` fields one by one to confirm whether all batch jobs are successful.

```sh
curl -X GET 'https://localhost/api/v1/audits/job-logs/?creator__name=zhangsan&order=-date_start&limit=100' \
    -H 'Authorization: Bearer <token>' \
    -H 'X-JMS-ORG: <organizationID>'
```

> For complete integration scenarios, please refer to: [Practical Case: One-click Audit of External Systems](../examples/one_click_audit.md)
