# Session authentication

## Overview
Session authentication is suitable for scenarios where you have logged in through a web page. After the browser successfully logs in, `jms_sessionid` will exist in the cookie. Subsequent requests only need to be in the same session/or manually attach the cookie to complete the authentication.

## Usage
- The front end or script reads `jms_sessionid` from the existing login context.
- When making a request, add: `Cookie: jms_sessionid=<value>` in the request header.
- No additional tokens or signatures are required.

**Request Example:**

```python
import requests

JMS_URL = 'https://demo.jumpserver.org'
SESSIONID = 'your_jms_sessionid'  # Capture the packet after logging in with the browser or obtain it from the developer tools

headers = {
    'Cookie': f'jms_sessionid={SESSIONID}',
    'X-JMS-ORG': '00000000-0000-0000-0000-000000000002'
}
resp = requests.get(f'{JMS_URL}/api/v1/users/users/', headers=headers)
resp.raise_for_status()
print(resp.json())
```

**Notes**

- Session authentication relies on server-side Session storage, has an expiration time, and is suitable for interactive or short-term scripts.
- It is recommended to use Token / Private Token / Access Key in automated long-term tasks.
- If cross-domain or third-party script calls are required, please consider using other authentication methods to avoid browser security restrictions.
