# Private Token 认证

## 概述
Private Token（永久 Token）不会自动过期，适合长期脚本或集成。使用时在请求头中：`Authorization: Token <private_token>`。

## 获取方式
在 JumpServer 服务器管理容器/主机执行：
```sh
docker exec -it jms_core /bin/bash
cd /opt/jumpserver/apps
python manage.py shell
from users.models import User
u = User.objects.get(username='admin')
u.create_private_token()
u.private_token  # 若已存在可直接读取
```
复制得到的 `private_token`。

## 使用方式

**请求示例：**

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

- **使用案例：**

场景：运维平台的夜间定时审计任务使用永久 Token（无需担心过期中断），拉取当前组织内已过期但仍处于激活状态的账号清单，生成账号安全日报。

```sh
curl "https://demo.jumpserver.org/api/v1/users/users/?is_expired=true&is_active=true&limit=20" \
    -H 'Authorization: Token <private_token>' \
    -H 'Content-Type: application/json' \
    -H 'X-JMS-ORG: <组织ID>'
```

**注意事项**

- Private Token 拥有与创建者相同权限，务必妥善保管。
- 建议定期轮换并撤销不再使用的 Token。