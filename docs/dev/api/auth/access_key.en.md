# Access Key authentication

## Overview
Access Key uses the request signature mechanism (HMAC-SHA256) and does not require a username or password. Suitable for third-party integration and automated calls to avoid directly exposing account credentials.

## Get Access Key
Create or view existing `AccessKeyID` and `AccessKeySecret` in the API Key list of the web interface.

## Usage

**Request Example:**

```python
# pip install requests httpsig
import requests, datetime, json
from httpsig.requests_auth import HTTPSignatureAuth

API_URL = 'https://demo.jumpserver.org'
ORG_ID = '00000000-0000-0000-0000-000000000002'
ACCESS_KEY_ID = 'AccessKeyID'
ACCESS_KEY_SECRET = 'AccessKeySecret'

def get_auth(key_id, secret):
    signature_headers = ['(request-target)', 'accept', 'date']
    return HTTPSignatureAuth(key_id=key_id, secret=secret, algorithm='hmac-sha256', headers=signature_headers)

def get_user_info():
    url = API_URL + '/api/v1/users/users/'
    gmt_form = '%a, %d %b %Y %H:%M:%S GMT'
    headers = {
        'Accept': 'application/json',
        'X-JMS-ORG': ORG_ID,
        'Date': datetime.datetime.utcnow().strftime(gmt_form)
    }
    auth = get_auth(ACCESS_KEY_ID, ACCESS_KEY_SECRET)
    r = requests.get(url, auth=auth, headers=headers)
    r.raise_for_status()
    print(json.dumps(r.json(), indent=2, ensure_ascii=False))

if __name__ == '__main__':
    get_user_info()
```

- **Use Case:**

Scenario: The nightly reconciliation script of the operations platform is inconvenient to save the account password. Instead, the Access Key signature method is used to call the interface to check whether the account of the specified user zhangsan is still valid.

```sh
API_URL='https://demo.jumpserver.org'
KEY_ID='<AccessKeyID>'
KEY_SECRET='<AccessKeySecret>'
REQUEST_TARGET='get /api/v1/users/users/?username=zhangsan'
DATE=$(date -u '+%a, %d %b %Y %H:%M:%S GMT')

SIGNING_STRING="(request-target): ${REQUEST_TARGET}
accept: application/json
date: ${DATE}"

SIGNATURE=$(printf '%s' "$SIGNING_STRING" | openssl dgst -sha256 -hmac "$KEY_SECRET" -binary | base64)

curl "${API_URL}/api/v1/users/users/?username=zhangsan" \
  -H 'Accept: application/json' \
  -H "Date: ${DATE}" \
  -H 'X-JMS-ORG: <organizationID>' \
  -H "Authorization: Signature keyId=\"${KEY_ID}\",algorithm=\"hmac-sha256\",headers=\"(request-target) accept date\",signature=\"${SIGNATURE}\""
```

**Notes**

- The Access Key should limit the visible range and be revoked and regenerated if necessary.
- Use HTTPS transmission to avoid man-in-the-middle attacks.
- The Date header must be in GMT format and consistent with the signature header list.
