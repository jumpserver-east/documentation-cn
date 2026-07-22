# 参数说明

!!! warning "注意"
    - 修改 /opt/jumpserver/config/config.txt 配置文件后需要重启 JumpServer 服务以生效。

## 1 Core 参数说明

!!! tip ""
    - Core 参数：

|  参数名称   | 可选项  | 说明  |
|  :-----  |  :-----  | :-----  |
| SECRET_KEY  | - | 用于对敏感字段进行加解密的 Key |
| BOOTSTRAP_TOKEN | - | 用于组件向 Core 服务注册使用的 Token |
| DEBUG  | true <br> false | Debug 模式，如果开启页面请求 API 报错时会显示更多信息 |
| DEBUG_DEV  | true <br> false | Debug 开发模式，如果开启后端日志会显示更多信息 |
| LOG_LEVEL | DEBUG <br> INFO <br> WARNING <br> ERROR <br> CRITICAL | 日志级别 |
| DB_ENGINE | postgresql/mysql | 数据库引擎 |
| DB_NAME | - | 数据库名 |
| DB_HOST | - | 数据库地址 |
| DB_PORT | - | 数据库端口	 |
| DB_USER| - | 数据库用户 |
| DB_PASSWORD | - | 数据库用户密码 |
| DB_USE_SSL | true <br> false | 数据库启用 SSL 方式 |
| REDIS_HOST | - | Redis 地址 |
| REDIS_PORT | - | Redis 端口 |
| REDIS_PASSWORD | - | Redis 密码 |
| REDIS_USE_SSL | true <br> false | Redis 启用 SSL 方式 |
| REDIS_SSL_KEY | - | Redis SSL Key |
| REDIS_SSL_CERT | - | Redis SSL Cert 证书 |
| REDIS_SSL_CA | - | Redis SSL CA Cert 证书 |
| REDIS_SSL_REQUIRED | - | Redis SSL 证书是否必须 |
| REDIS_SENTINEL_HOSTS | - | Redis 哨兵地址（多个地址使用 / 分割） |
| REDIS_SENTINEL_PASSWORD | - | Redis 哨兵密码 |
| REDIS_SENTINEL_SOCKET_TIMEOUT | - | Redis 哨兵 Socket 超时时间 |
| REDIS_DB_CELERY | 0-15	| Redis 库编号，Celery 任务使用 |
| REDIS_DB_CACHE | 0-15 | Redis 库编号，缓存使用 |
| REDIS_DB_SESSION | 0-15 | Redis 库编号，用户 Session 使用 |
| REDIS_DB_WS | - | Redis 库编号，WebSocket 使用 |
| TOKEN_EXPIRATION | - | 通过 API 创建用户 Token 的有效期 <br> # 如果配置为空或者0，则默认值为 3600 |
| DEFAULT_EXPIRED_YEARS | - | 创建资源的默认过期年份，比如：授权规则 <br> # 不允许修改 |
| SESSION_COOKIE_DOMAIN | - | 用户 Session Cookie 域，比如：fit2cloud.com |
| CSRF_COOKIE_DOMAIN | - | 用户 CSRF Cookie 域，默认与 SESSION_COOKIE_DOMAIN 保持一致 |
| SESSION_COOKIE_NAME_PREFIX | - | 用户 Session Cookie 名称的前缀 <br> # 如果配置了 SESSION_COOKIE_DOMAIN 参数，会使用 `.` 前的值作为默认值，比如：fit2cloud |
| SESSION_COOKIE_AGE | - | 用户 Session Cookie 的有效期 |
| SESSION_EXPIRE_AT_BROWSER_CLOSE | true <br> false | 用户 Session 在浏览器关闭后过期 |
| CONNECTION_TOKEN_EXPIRATION | >= 5 * 60 | 有效期内 ConnectionToken 只能使用一次	 |
| CONNECTION_TOKEN_EXPIRATION_MAX | - | 有效期内 ConnectionToken 可以多次使用 |
| CONNECTION_TOKEN_REUSABLE | true <br> false | ConnectionToken 是否可以多次使用 |
| AUTH_CUSTOM | true <br> false | 开启自定义用户认证 |
| AUTH_CUSTOM_FILE_MD5 | - | 自定义用户认证的文件 md5 值 |
| MFA_CUSTOM | true <br> false | 开启自定义 MFA 认证 |
| MFA_CUSTOM_FILE_MD5 | - | 自定义 MFA 认证的文件 md5 值 |
| AUTH_TEMP_TOKEN | true <br> false | 开启临时密码功能 |
| LOGIN_REDIRECT_TO_BACKEND | Direct（直接进入内部登录页面） <br> OpenID <br> CAS <br> SAML2 <br> OAuth2 的服务提供商名称（系统设置） | 开启第三方认证后，不出现倒计时跳转页面直接跳转到认证服务，比如：OpenID |
| LOGIN_REDIRECT_MSG_ENABLED | true <br> false | 开启第三方跳转倒计时页面 |
| SYSLOG_ADDR | - | SysLog 服务地址 |
| SYSLOG_FACILITY | - | SysLog FACILITY |
| SYSLOG_SOCKTYPE | - | SysLog SockType |
| PERM_EXPIRED_CHECK_PERIODIC | - | 校验过期的资产授权规则并过期用户授权树的周期 |
| LANGUAGE_CODE | zh <br> en <br> ja | 语言 |
| TIME_ZONE | - | 时区 |
| SESSION_COOKIE_SECURE | true <br> false | 用户 Session Cookie 安全模式，开启后只允许在 https 协议下发送 |
| CSRF_COOKIE_SECURE | true <br> false | 用户 CSRF Token 安全模式，开启后只允许在 https 协议下发送 |
| REFERER_CHECK_ENABLED | true <br> false | 开启 REFERER 校验 |
| CSRF_TRUSTED_ORIGINS | - | CSRF 同源信任，多个地址使用 `,` 分割 |
| SESSION_ENGINE | - | 用户 Session 引擎 |
| SESSION_SAVE_EVERY_REQUEST | true <br> false | 每个请求都要保存用户 Session |
| SESSION_EXPIRE_AT_BROWSER_CLOSE_FORCE | true <br> false | 浏览器关闭后强制过期用户 Session 会话 |
| SERVER_REPLAY_STORAGE | - | 服务端录像存储 <br> 比如：<br> {<br>    'TYPE': 's3',<br>    'BUCKET': '',<br>    'ACCESS_KEY': '',<br>    'SECRET_KEY': '',<br>    'ENDPOINT': ''<br>}  <br> # 组件上传录像到 Core 服务，Core 自动上传到配置的对象存储服务 |
| CHANGE_AUTH_PLAN_SECURE_MODE_ENABLED | true <br> false | 改密计划安全模式 <br> 启用后，不支持用户自己改自己； <br> 禁用后，支持自己改自己； <br> 比如 root 改 root |
| SECURITY_VIEW_AUTH_NEED_MFA | true <br> false | 需要校验 MFA |
| SECURITY_DATA_CRYPTO_ALGO | aes_ecb <br> aes_gcm <br> aes <br> gm_sm4_ecb <br> gm | 数据加密算法 |
| GMSSL_ENABLED | true <br> false | 开启国密算法（数据加密算法） <br> SECURITY_DATA_CRYPTO_ALGO <br> GMSSL_ENABLED <br> # 如果同时配置，优先使用 SECURITY_DATA_CRYPTO_ALGO |
| OPERATE_LOG_ELASTICSEARCH_CONFIG | - | 操作日志"变更字段"的存储ES配置 <br>比如：<br>{<br>    "INDEX": "",<br>    "HOSTS": "",<br>    "OTHER": "",<br>    "IGNORE_VERIFY_CERTS": "",<br>    "INDEX_BY_DATE": "",<br>    "DOC_TYPE": ""<br>} |
| MAGNUS_ORACLE_PORTS | - | Magnus 组件需要监听的 Oracle 端口范围 |
| APPLET_DOWNLOAD_HOST | - | Applet 等软件的下载地址 |
| FTP_FILE_MAX_STORE | - |  FTP 文件上传下载备份阈值，单位(M)，当值<=0时，不备份文件 |
