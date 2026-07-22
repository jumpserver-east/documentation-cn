# Interactive commands

!!! warning "Improper operation will result in data loss, please confirm carefully before operation"

## 1 How to operate

!!! tip ""
    ```sh
    docker exec -it jms_core bash
    cd /opt/jumpserver/apps
    python manage.py shell
    ```
    ```python
    # Before operating the new version, you need to switch to the corresponding organization., Default is Default
    from orgs.models import *
    Organization.objects.all()
    org = Organization.objects.get(name='Default')
    org.change_to()
    ```

!!! tip ""
    - Select the interactive command object to view

    === "User"
        ```python
        from users.models import *

        # User
        User.objects.all()
        User.objects.count()  # Quantity

        # Specify user query
        user = User.objects.get(username = 'admin')

        # Query user email
        user.email

        # Modify user email
        user.email='test@jumpserver.org'

        # Change password
        user.reset_password('test01')

        # Save changes
        user.save()

        # Delete user MFA key
        user.otp_secret_key=''
        user.save()

        # Create new user
        User.objects.create(name = 'test user', username = 'test', email = 'test@jumpserver.org')

        # Test whether the user has the same name, Create if it does not exist
        User.objects.get_or_create(name = 'test user', username = 'test', email = 'test@jumpserver.org')

        user = User.objects.get(username = 'test')
        user.delete()

        # A more elegant way to delete
        User.objects.all().filter(username='test').delete()
        ```
        ```python
        # User group
        UserGroup.objects.all()
        UserGroup.objects.count()

        # Create user group
        UserGroup.objects.create(name = 'Test')
        group = UserGroup.objects.get(name = 'Test')

        # Add user to user group
        user = User.objects.get(username='test')
        group.users.add(user)
        group.save()

        # Delete user from user group
        user = User.objects.get(username='test')
        group.users.remove(user)
        group.save()

        # Delete user group
        UserGroup.objects.all().filter(name='Test').delete()
        ```
    
    === "Asset"
        ```python
        from assets.models import *

        # assets
        Asset.objects.all()
        Asset.objects.count()

        # create
        asset = Asset.objects.create(name = 'test', address = '172.16.0.1')

        # Delete
        asset = Asset.objects.get(name = 'test')
        asset.delete()
        ```
        ```python
        # node
        Node.objects.all()
        Node.objects.count()

        node = Node.objects.get(value = 'Test')

        asset = Asset.objects.get(name = 'test')
        # Add assets to nodes
        node.assets.add(asset)

        # Remove assets from node
        node.assets.remove(asset)

        node.delete()
        ```


## 2 Data decryption
!!! tip ""

    === "System User"
        ```sh
        docker exec -it jms_core bash
        cd /opt/jumpserver/apps
        python manage.py shell
        ```
        ```python
        from accounts.models import Account
        # 81aef7ac-e432-4d1b-aaf5-a3bc37c2b230 I want to check your account for you id, in web There is this field in the account details on the page
        a = Account.objects.get(id='81aef7ac-e432-4d1b-aaf5-a3bc37c2b230')
        ```
        ```python
        a.secret_type
        ```
        ```nginx
        Out[4]: 'password'
        ```
        ```python
        a.secret
        ```
        ```nginx
        Out[6]: '-----BEGIN RSA PRIVATE KEY-----
        MIIEpAIBAAKCAQEAxg4C1KKD5mz+3arCKETugJggR4HzEvIjAutKv+zZwAYm5SbB
        3IGXoEzdXbk/9u1btyTGbmTpKubsJh5MeGlHWExqzA2n9NsC/3hYenjwm1OP1Vhc
        bGZYnZTqbUGTiWiRhtXUCOC2yzgSdLiCLGS5XdIVEhCO3AvWZCvauhEQYu3PLlUN
        Xuc7JZBLGrZ+YVo87b+AvwpnAX2igWRTHdAlP0hL+MWoN3lzba90Jox1zJNeyZ64
        M/u1TsiMGGWs/H35SqpH9jerCl5+1Mqw5oryidYApuNOilN8ucYa6XDueweEXkHk
        oY8QxwaC8GFLZErb0Ov8lzEhlpALsCYgekiecQIDAQABAoIBAQC5W/OaPl9kES6X
        F3GPbrQo9jd/tUdhu+y4lq3m4i0JYriUTqmxTjgydr3XMcGDwLHNvkVYnGj9FhJ9
        um2nZCC5qwto3n8K1s8DegaU2QuXLX64FXKqoT8efLjKeE00lQFeSFGh3W4208uy
        Idzy33H9NNkzhvutRgboyYJ0EfRcIL/wyxc0ndMKt/bVYH8T14aWViVRF7OyiIkd
        eFx7tdnVscBSujNX027ycmjElwRf9TPNFUFwF5XGf3xwWjPmBNWr91XxPcF9VecG
        gyd9qxNd12YYGcX4SR6V0p+36Av+rZoHB0405b/ZncmevSStUCu6fTtQYwdCZj0p
        PgTradABAoGBAOVjCRleXfO9OVc4Y7sM2+1i/So5dmp66foC76j6CDKzztA2b5FX
        tduNhoObeNYdnV32WvW3/xXcFWFnbWf0Eymx2DMuxMfWlTvM0InParq0TpeVMWMP
        uxW+7YNZ9IWuuLMs3jfY1lRQBUgVlcb0zA7tjWZO/n/mKW5Fd2rItvmRAoGBAN0I
        YzFGEPoKgGqYme58KpebM1jt1XoyLtbH1ygRvaPlnLfPDBsBhrqLCyR3+oK0kFlM
        f7Neqo86hCQ/aqVC1lMu2o3htg52b1Mj2T0YUTNsPTwx+8lHciRnqJZytHRjwfFC
        4UySAzWKDDQZcIQGTAsdoXngkAZFITMBZBdRz+bhAoGAfytphvvvIEq+eGFVwQR/
        BNtFOVyEDsI35xgrn8WGN/3BYWNcdPpoYuDSOzI9So8+iDIk+WbZb1gFLmv1lpUU
        7p+fGbkK9TM8ptuEnXI1XG7Lx3O53o6BDKw95vz+98IGuabdR57aLAH0+6Kj15ot
        avU92ANhSqziOTUf4D6IWlECgYAe+kr0n+5HLOuchPCl9O7/Ongy0Xpm2tunrHBi
        JEJg0xBoznLS5h7gzBXusYYBhY7phQgsumrLEhdtARpQORa/Q8TLt8ONOVoW2+JZ
        ZqwSuevHIPY52nKL2Z9OHptd6JFI3+e1lI0wlr1pG9uiFUPZFvkHnMpypoOlo19E
        yWmK4QKBgQCVmBTLSA+M3WJqDBK2Z6lUaDCaAjwn2Q5RSq2B/lLjzaod6WYWyecY
        NASeo6CC4fxOCfMJN1DT5CLyW4XpRk3GeR4QKSfFkwD2yRqk+7opm8PppdMuLZKU
        LQMMI90AWvU3Cx9aAbl1bLSIT0qRoc5FGwmLEL12yDBZA2l3vYhnaw==
        -----END RSA PRIVATE KEY-----'
        ```
    
    === "System Settings Field"
        ```sh
        docker exec -it jms_core bash
        cd /opt/jumpserver/apps
        python manage.py shell
        ```
        ```python
        from settings.models import Setting

        s = Setting.objects.get(name='EMAIL_HOST_PASSWORD')
        ```
        ```python
        s.cleaned_value
        ```
        ```nginx
        Out[9]: '123456'
        ```
