# Quick start
## 1 Install JumpServer
!!! tip ""
    - Supports mainstream Linux distributions (based on Debian/RedHat, including domestic operating systems)
    - It is recommended to use [offline installation package method](installation/setup_linux_standalone/offline_install.md) to deploy JumpServer

!!! info "After successful installation, log in to JumpServer via browser"
    ```sh
    Address: http://<JumpServerserverIPAddress>:<Service running port>
    Username: admin
    Password: ChangeMe
    ```

## 2 Asset Management
### 2.1 Preparation
!!! tip ""
    - Prepare two test assets and a database to verify functionality.

!!! tip ""

    |   IP address     |    hostname       |    port    | operating system         |  Admin user    |    Password       |
    | ------------ | --------------- | ---------- | ---------------- |--- ---------- |--- ---------- |
    | 172.16.80.11 |    test_ssh01   |     22     |     Centos 7     |      root     |  Test2020.L   |
    | 172.16.80.21 |    test_rdp01   |    3389    |    Windows 10    | administrator |  Test2020.W   |
    | 172.16.80.31 |   test_mysql01  |    3306    |      MySQL 5     |      root     |  Test2020.M   |

!!! warning "attention"
    - If Windows assets need to perform automated tasks such as `update asset` information, `connectability test`, etc., you need to perform [Windows SSH Settings](guide/asset_requirements/windows_ssh.md) first. This is not required to log in to Windows assets.
    - MySQL applications need permission to authorize remote access to `Core` and `KoKo` [MySQL application requirements](guide/asset_requirements/mysql.md)

### 2.2 Edit asset tree
!!! tip ""
    - Click `Asset Management` - `Asset List` on the left side of the page, right-click on the root node `Default` and create three new nodes: `SSH Server`, `RDP Server` and `Database`.
!!! tip ""
    - The asset tree style is as follows:

    ```
    Default
    ├─ SSH Server
    └─ RDP Server
    └─ DB Server
    ```

!!! warning "attention"
    - The root node `Default` cannot be renamed. Right-click the node to `add`, `delete` and `rename` nodes, as well as perform asset-related operations.  

### 2.3 Create assets
!!! tip ""
    - Click `Asset Management` - `Asset List` - `Host` - `Create` on the left side of the page to create a Linux server, and during the asset creation process, create a privileged user. The content is the `Administrator User` and `Password` in the form above.
    - The same goes for creating Windows assets.

!!! tip ""
    - Create a Linux asset style as follows:

!!! tip ""

    | Name        | IP/host      | Asset platform | node          | protocol group    | Account list   |
    | ---------- | ------------ | ------- | ------------ | -------  | -------- |
    | test_ssh01 | 172.16.80.11 | Linux   | /Default/SSH Server | ssh 22 | add |

!!! tip ""
    - Add login asset user style as follows:

!!! tip ""

    | Name              | Username | privileged user | Ciphertext type     | Password          |
    | ----------------- | ---- | ------- | ---------- | ---------------- |
    | 172.16.80.11_root | root  | Yes | Password |Test2020.L |

!!! warning "attention"
    - `Name` cannot have the same name. You can choose either `Password` or `Key`. Some assets are not allowed to pass `Password` authentication and you can use `Private Key` authentication instead.  
    - `Privileged user` only supports the `SSH` protocol and is used for automated tasks such as asset `connectability testing`, `push user`, `batch password change`, etc.
    - After filling in the asset creation information and saving it, refresh the web page every few seconds. The connectable icon of the `ssh` protocol asset will be displayed in `green`, and the `hardware information` will be displayed.  
    - If the `Connectable` icon is `Yellow` or `Red`, you can click the `Name` of the `Asset`, on the right `Quick Modify` - `Test Connectability` click the `Test` button and handle it according to the error prompts.  
    - The connected `Linux` assets require the `python` component, and the version is greater than or equal to `2.6`. Assets such as `Ubuntu` do not allow remote `ssh` login by the `root` user by default. Please handle it by yourself. `Windows` assets require manual installation of `OpenSSH Server`.
    - If the asset cannot be connected normally, please check whether the `username` and `password` of the privileged user are correct and whether the `privileged user` can use `SSH` to correctly log in to the asset host from the `JumpServer` host.

### 2.4 Create database application
!!! tip ""
    - Click `Asset Management` - `Asset List` - `Create` - `Select MySQL database under Database` on the left side of the page.

!!! tip ""
    - Create a MySQL database application style as follows:

!!! tip ""

    | Name         | Address          | node               | database | protocol group      | Account list  |
    | ------------ | ------------ | ------------------ | ----- | ----------- | -------- |
    | test_mysql01 | 172.16.80.31 | /Default/DB Server | test  | mysql:3306  | add     |

!!! tip ""
    - Add the login database user style as follows:

!!! tip ""

    |        Name       | Username | privileged user | Ciphertext type |    Password    |
    | ----------------- | ----- | ------ | -------  | ---------- |
    | 172.16.80.23_root | root  | root   | Password     | Test2020.M |

!!! warning "attention"
    - Name, host, and database options are required.

## 3 Create authorization rules
!!! tip ""
    - Click `Permission Management` - `Asset Authorization` - `Create` on the left side of the page to create an authorization.
    - The authorization process for Windows assets and MySQL databases is the same as the following.

!!! tip ""
    - Create a login authorization rule (for example, Linux assets) with the following style:

!!! tip ""

    | Name             | User                 | User group | assets                     | node | Account number                                  | action                  |
    | ---------------- | -------------------- | ----- | ------------------------ | --- | ----------------------------------------- | -------------------- |
    | admin_ssh01 | Administrator(admin) |   -    | test_ssh01(172.16.80.11) |  -   | All accounts                  | :material-check: all |

!!! warning "attention"
    - `Name`, the authorized name, cannot be repeated.  
    - Choose between `User` and `User Group`. It is not recommended to select `User` and `User Group`.  
    - Choose one of `Assets` and `Nodes`. Selecting `Nodes` will include all `Assets` under `Nodes`.  
    - `Account`, `Account` is the `authentication credential` for connecting assets.  
    - There is a one-to-one relationship between `User (Group)` and `Asset (Node)`, so when you have different types of assets `Linux` and `Windows`, you should create `Authorization Rules` for `Linux` assets and `Windows` assets respectively.  

## 4 User login
!!! tip ""
    - Click `Web Terminal` in the upper right corner of the page to connect assets.

!!! warning "attention"
    - Users can only see the assets they have been authorized by the administrator. If there are no assets after logging in, please contact the administrator for confirmation.

## 5 system settings
!!! tip ""
    - Click `System Settings` in the upper right corner of the page to configure.

### 5.1 Basic settings
!!! tip ""

    | Name          | Example                        | Remarks                                         |
    | ------------ | --------------------------- | -------------------------------------------- |
    | Current site URL   | https://demo.jumpserver.org | If not set, the email received address will be `http://localhost` |
    | User guide URL   |                                                          | Users can see this `hyperlink` when logging in for the first time and do not need to set it.      |
    | Forgot password URL   |                                                          | Uses LDAP, OPENID and other external authentication systems, which can be customized  |

### 5.2 Email settings
!!! tip ""
    - We support email configuration via `SMTP` or `EXCHANGE`.

=== "SMTP"
    !!! tip ""

        | Name | Example | Remarks |
        | ---------- | ---------------- | ---------------------------------- |
        | SMTP host   | smtp.qq.com      | SMTP server provided by service provider             |
        | SMTP port   | 25               | Usually `25`                         |
        | SMTP account   | **********@qq.com | Usually `user@domain.com`            |
        | SMTP password   | **************** | You need to re-enter the password every time you `test the connection`    |
        | Use SSL    | [ ]              | If the port uses `465`, this must be checked      |
        | Use TLS    | [ ]              | If the port uses `587`, this must be checked      |
        | sender     | **********@qq.com | `Test Connection` must be entered                |
        | Topic prefix   | [JMS]            | The title of the email. The received email starts with `[JMS]` |
        | Test recipient | **********@qq.com | Test connection required                         |

    !!! warning "attention"
        - You cannot check `Use SSL` and `Use TLS` at the same time.


## 6 Commonly used function operations
!!! tip ""
    - **[Upload and download via SFTP][Upload and download via SFTP]**
    - **[Windows upload and download][Windows upload and download]**
    - **[Restrict IP login][Restrict IP login]**
    - **[Managed database application][Managed database application]**
    - **[Redis database management][Redis database management]**


[Upload and download via SFTP]: https://kb.fit2cloud.com/?p=115
[Windows upload and download]: https://kb.fit2cloud.com/?p=87
[Restrict IP login]: https://kb.fit2cloud.com/?p=199
[Redis database management]: https://kb.fit2cloud.com/?p=91
[Managed database application]: https://kb.fit2cloud.com/?p=79
