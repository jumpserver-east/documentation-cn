# Private Token authentication

## Overview
Private Token (permanent Token) does not automatically expire and is suitable for long-term scripts or integrations. When used in the request header: `Authorization: Token <private_token>`.

## How to get it
Execute on the JumpServer server management container/host:
```sh
docker exec -it jms_core /bin/bash
cd /opt/jumpserver/apps
python manage.py shell
from users.models import User
u = User.objects.get(username='admin')
u.create_private_token()
u.private_token  # If it already exists, it can be read directly.
```
Copied `private_token`.

## Usage

**Request Example:**

**CURL**
```sh
curl https://demo.jumpserver.org/api/v1/users/users/ \
    -H 'Authorization: Token 937b38011acf499eb474e2fecb424ab3' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```
**Python**
```python
import requests, json

API_URL = 'https://demo.jumpserver.org'
PRIVATE_TOKEN = '937b38011acf499eb474e2fecb424ab3'
ORG_ID = '00000000-0000-0000-0000-000000000002'

def get_user_info():
    url = API_URL + '/api/v1/users/users/'
    headers = {
        'Authorization': 'Token ' + PRIVATE_TOKEN,
        'X-JMS-ORG': ORG_ID
    }
    r = requests.get(url, headers=headers)
    r.raise_for_status()
    print(json.dumps(r.json(), indent=2, ensure_ascii=False))

if __name__ == '__main__':
    get_user_info()
```

- **Use Case:**

Scenario: The nightly scheduled audit task of the operations platform uses a permanent token (no need to worry about expiration and interruption), pulls the list of accounts that have expired but are still active in the current organization, and generates an account security daily report.

```sh
curl "https://demo.jumpserver.org/api/v1/users/users/?is_expired=true&is_active=true&limit=20" \
    -H 'Authorization: Token <private_token>' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: <organizationID>'
```

**Notes**

- Private Token has the same permissions as the creator and must be kept properly.
- It is recommended to regularly rotate and revoke tokens that are no longer used.
