# Configuration Parameters

!!! warning "Attention"
    - Restart JumpServer after changing `/opt/jumpserver/config/config.txt` so the new configuration takes effect.

## 1 Core Parameters

!!! tip ""
    - Core parameters:

| Parameter | Options | Description |
| :--- | :--- | :--- |
| SECRET_KEY | - | Key used to encrypt and decrypt sensitive fields. |
| BOOTSTRAP_TOKEN | - | Token used when components register with the Core service. |
| DEBUG | true <br> false | Debug mode. When enabled, the page displays more information when an API request fails. |
| DEBUG_DEV | true <br> false | Debug development mode. When enabled, backend logs contain more information. |
| LOG_LEVEL | DEBUG <br> INFO <br> WARNING <br> ERROR <br> CRITICAL | Log level. |
| DB_ENGINE | postgresql/mysql | Database engine. |
| DB_NAME | - | Database name. |
| DB_HOST | - | Database address. |
| DB_PORT | - | Database port. |
| DB_USER | - | Database user. |
| DB_PASSWORD | - | Database user password. |
| DB_USE_SSL | true <br> false | Enable an SSL connection to the database. |
| REDIS_HOST | - | Redis address. |
| REDIS_PORT | - | Redis port. |
| REDIS_PASSWORD | - | Redis password. |
| REDIS_USE_SSL | true <br> false | Enable an SSL connection to Redis. |
| REDIS_SSL_KEY | - | Redis SSL key. |
| REDIS_SSL_CERT | - | Redis SSL certificate. |
| REDIS_SSL_CA | - | Redis SSL CA certificate. |
| REDIS_SSL_REQUIRED | - | Whether a Redis SSL certificate is required. |
| REDIS_SENTINEL_HOSTS | - | Redis Sentinel addresses. Separate multiple addresses with `/`. |
| REDIS_SENTINEL_PASSWORD | - | Redis Sentinel password. |
| REDIS_SENTINEL_SOCKET_TIMEOUT | - | Redis Sentinel socket timeout. |
| REDIS_DB_CELERY | 0-15 | Redis database number used by Celery tasks. |
| REDIS_DB_CACHE | 0-15 | Redis database number used for cache. |
| REDIS_DB_SESSION | 0-15 | Redis database number used for user sessions. |
| REDIS_DB_WS | - | Redis database number used for WebSocket. |
| TOKEN_EXPIRATION | - | Validity period of a user token created through the API.<br>If empty or `0`, the default is `3600`. |
| DEFAULT_EXPIRED_YEARS | - | Default expiration period in years for created resources, such as permission rules.<br>Do not change this parameter. |
| SESSION_COOKIE_DOMAIN | - | User session cookie domain, for example `fit2cloud.com`. |
| CSRF_COOKIE_DOMAIN | - | User CSRF cookie domain. It defaults to `SESSION_COOKIE_DOMAIN`. |
| SESSION_COOKIE_NAME_PREFIX | - | Prefix of the user session cookie name.<br>If `SESSION_COOKIE_DOMAIN` is configured, the value before the first `.` is used by default, for example `fit2cloud`. |
| SESSION_COOKIE_AGE | - | User session cookie validity period. |
| SESSION_EXPIRE_AT_BROWSER_CLOSE | true <br> false | Expire the user session when the browser closes. |
| CONNECTION_TOKEN_EXPIRATION | >= 5 * 60 | A ConnectionToken can be used only once during this validity period. |
| CONNECTION_TOKEN_EXPIRATION_MAX | - | A ConnectionToken can be used multiple times during this validity period. |
| CONNECTION_TOKEN_REUSABLE | true <br> false | Whether a ConnectionToken can be used multiple times. |
| AUTH_CUSTOM | true <br> false | Enable custom user authentication. |
| AUTH_CUSTOM_FILE_MD5 | - | MD5 value of the custom user authentication file. |
| MFA_CUSTOM | true <br> false | Enable custom MFA authentication. |
| MFA_CUSTOM_FILE_MD5 | - | MD5 value of the custom MFA authentication file. |
| AUTH_TEMP_TOKEN | true <br> false | Enable temporary passwords. |
| LOGIN_REDIRECT_TO_BACKEND | Direct (open the internal login page directly) <br> OpenID <br> CAS <br> SAML2 <br> Name of an OAuth2 service provider configured in System Settings | After third-party authentication is enabled, redirect directly to the authentication service without displaying the redirect countdown page. For example, use `OpenID`. |
| LOGIN_REDIRECT_MSG_ENABLED | true <br> false | Enable the third-party authentication redirect countdown page. |
| SYSLOG_ADDR | - | Syslog service address. |
| SYSLOG_FACILITY | - | Syslog facility. |
| SYSLOG_SOCKTYPE | - | Syslog socket type. |
| PERM_EXPIRED_CHECK_PERIODIC | - | Interval for checking expired asset permission rules and expiring user permission trees. |
| LANGUAGE_CODE | zh <br> en <br> ja | Language. |
| TIME_ZONE | - | Time zone. |
| SESSION_COOKIE_SECURE | true <br> false | Secure mode for user session cookies. When enabled, cookies are sent only over HTTPS. |
| CSRF_COOKIE_SECURE | true <br> false | Secure mode for user CSRF tokens. When enabled, tokens are sent only over HTTPS. |
| REFERER_CHECK_ENABLED | true <br> false | Enable Referer validation. |
| CSRF_TRUSTED_ORIGINS | - | Trusted CSRF origins. Separate multiple addresses with `,`. |
| SESSION_ENGINE | - | User session engine. |
| SESSION_SAVE_EVERY_REQUEST | true <br> false | Save the user session on every request. |
| SESSION_EXPIRE_AT_BROWSER_CLOSE_FORCE | true <br> false | Force the user session to expire after the browser closes. |
| SERVER_REPLAY_STORAGE | - | Server-side session recording storage.<br>For example:<br>{<br>    'TYPE': 's3',<br>    'BUCKET': '',<br>    'ACCESS_KEY': '',<br>    'SECRET_KEY': '',<br>    'ENDPOINT': ''<br>}<br>Components upload recordings to Core, and Core uploads them to the configured object storage service. |
| CHANGE_AUTH_PLAN_SECURE_MODE_ENABLED | true <br> false | Secure mode for secret change plans.<br>When enabled, users cannot change their own secrets.<br>When disabled, users can change their own secrets, for example `root` changing `root`. |
| SECURITY_VIEW_AUTH_NEED_MFA | true <br> false | Require MFA verification. |
| SECURITY_DATA_CRYPTO_ALGO | aes_ecb <br> aes_gcm <br> aes <br> gm_sm4_ecb <br> gm | Data encryption algorithm. |
| GMSSL_ENABLED | true <br> false | Enable a GM cryptographic algorithm for data encryption.<br>If both `SECURITY_DATA_CRYPTO_ALGO` and `GMSSL_ENABLED` are configured, `SECURITY_DATA_CRYPTO_ALGO` takes precedence. |
| OPERATE_LOG_ELASTICSEARCH_CONFIG | - | Elasticsearch configuration used to store the `changed fields` of operation logs.<br>For example:<br>{<br>    "INDEX": "",<br>    "HOSTS": "",<br>    "OTHER": "",<br>    "IGNORE_VERIFY_CERTS": "",<br>    "INDEX_BY_DATE": "",<br>    "DOC_TYPE": ""<br>} |
| MAGNUS_ORACLE_PORTS | - | Oracle port range on which the Magnus component listens. |
| APPLET_DOWNLOAD_HOST | - | Download address for applets and other software. |
| FTP_FILE_MAX_STORE | - | Backup threshold for uploaded and downloaded FTP files, in MB. Files are not backed up when the value is less than or equal to `0`. |
