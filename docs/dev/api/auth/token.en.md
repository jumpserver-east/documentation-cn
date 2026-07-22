# Token authentication

## Overview
The one-time token can be exchanged for username and password, and can be used within the validity period. The request header is passed with `Authorization: Bearer <token>`.

## How to get it
**CURL**
```sh
curl -X POST http://localhost/api/v1/authentication/auth/ \
     -H 'Content-Type: application/json' \
     -d '{"username": "admin", "password": "admin"}'
```
The return contains the field `token`.


## Usage

**Request Example:**

**Python**
```python
# pip install requests
import requests, json

API_URL = 'https://demo.jumpserver.org'
USERNAME = 'admin'
PASSWORD = 'admin'
ORG_ID = '00000000-0000-0000-0000-000000000002'

def get_token(jms_url, username, password):
    url = jms_url + '/api/v1/authentication/auth/'
    data = {"username": username, "password": password}
    r = requests.post(url, json=data)
    r.raise_for_status()
    return r.json()['token']

def get_user_info(jms_url, token):
    url = jms_url + '/api/v1/users/users/'
    headers = {
        'Authorization': 'Bearer ' + token,
        'X-JMS-ORG': ORG_ID
    }
    r = requests.get(url, headers=headers)
    r.raise_for_status()
    print(json.dumps(r.json(), indent=2, ensure_ascii=False))

if __name__ == '__main__':
    tk = get_token(API_URL, USERNAME, PASSWORD)
    get_user_info(API_URL, tk)
```

**Notes**

- Token has an expiration date and needs to be reacquired regularly.
- Avoid hardcoding tokens in public repositories.

- **Use Case:**

Scenario: Before running the daily inspection script, use the dedicated operations account ops-robot to exchange for a one-time token, and immediately bring the token to call the interface to verify the availability.

```sh
# 1. Use a dedicated account in exchange for one-time use Token
TOKEN=$(curl -s -X POST https://jms.example.com/api/v1/authentication/auth/ \
     -H 'Content-Type: application/json' \
     -d '{"username": "ops-robot", "password": "Robot@2026"}' | jq -r '.token')

# 2. carry Token Call interface，Verify whether the certification is valid
curl -X GET https://jms.example.com/api/v1/users/users/ \
     -H 'Authorization: Bearer <token>' \
     -H 'X-JMS-ORG: <organizationID>'
```
