# Basic Operations

## 1 The jmsctl Command-Line Tool

The default JumpServer installation script is located at `<extracted-package-path>/jmsctl.sh`. JumpServer also provides the `jmsctl` command-line tool.

> Attention: You may delete the JumpServer installation archive after installation, but do not delete the extracted directory.

You can run `jmsctl` from any directory on the server. To run `./jmsctl.sh`, first change to the extracted installation directory.

### 1.1 Command Syntax

- `jmsctl [COMMAND]`
- `./jmsctl.sh [COMMAND]` (run from the extracted installation package directory)

### 1.2 Commands

| Command | Description |
| :--- | :--- |
| `jmsctl help` | Display help. |
| `jmsctl start` | Start all JumpServer service containers. |
| `jmsctl stop` | Stop all JumpServer service containers. |
| `jmsctl down` | Stop and remove all JumpServer service containers. |
| `jmsctl restart` | Restart all JumpServer service containers. |
| `jmsctl status` | Display the status of JumpServer service containers. |
| `jmsctl backup_db` | Back up the JumpServer database. |
| `jmsctl uninstall` | Uninstall JumpServer. This operation deletes all JumpServer data. Proceed with caution. |
| `jmsctl restore_db` | Restore the database from a backup SQL file. |

> Attention: The `jmsctl restart` and `jmsctl down` commands remove or recreate containers. The service is interrupted, but persistent and business data are not affected.

## 2 Modify Configuration Files

### 2.1 Configuration File Reference

The following example explains the default JumpServer parameters:

!!! tip ""
    ```ini
    # JumpServer configuration file example.
    # If you do not understand an option, leave it unchanged. The system supplies its value automatically.
    # For the complete parameter reference, see Configuration Parameters in this manual.

    ################################ Image Configuration ############################
    # Connections to docker.io can time out or be slow in mainland China.
    # Enable this option to use Huawei Cloud image acceleration.
    # This replaces DOCKER_IMAGE_PREFIX from previous versions.
    # DOCKER_IMAGE_MIRROR=1

    # Image pull policy: Always or IfNotPresent.
    # Always pulls the latest image every time. IfNotPresent pulls only when the image is not available locally.
    # IMAGE_PULL_POLICY=Always

    ############################# Installation Configuration ########################
    # JumpServer data persistence directory. Recordings and task logs are stored here by default.
    # Change it as required. Database (.sql) and configuration backups created during upgrades are also stored here.
    VOLUME_DIR=/data/jumpserver  # JumpServer persistent data directory

    # Encryption key. During migration, keep SECRET_KEY identical to the previous environment.
    # Do not use special characters.
    # (*) Warning: Keep this value secret.
    # (*) Never disclose SECRET_KEY.
    SECRET_KEY=********************************  # Example key used by the system to encrypt and decrypt data

    # Token used when components register with Core.
    # During migration, keep BOOTSTRAP_TOKEN identical to the previous environment.
    # Do not use special characters.
    # (*) Warning: Keep this value secret.
    # (*) Never disclose BOOTSTRAP_TOKEN.
    BOOTSTRAP_TOKEN=****************  # Example token used by other components to register with JumpServer

    # Log levels: INFO, WARN, ERROR.
    LOG_LEVEL=ERROR  # Set DEBUG temporarily for more details, but monitor the resulting log volume.

    # Network used by JumpServer containers. Change it if it conflicts with an existing network.
    DOCKER_SUBNET=192.168.250.0/24  # IP address range for JumpServer containers

    # IPv6 NAT. It normally does not need to be enabled.
    # Enabling this option when the host does not support IPv6 prevents the real client IP address from being detected.
    USE_IPV6=0
    DOCKER_SUBNET_IPV6=fc00:1010:1111:200::/64

    ################################ Database Configuration ##########################
    # Enter valid database information for an external database.
    # The system configures a built-in database automatically.
    #
    DB_ENGINE=postgresql  # Database type: mysql or postgresql
    DB_HOST=postgresql  # Database address. The value postgresql starts the built-in PostgreSQL container.
    DB_PORT=5432  # Database port
    DB_USER=postgres  # Database user
    DB_PASSWORD=********************  # Example database user password
    DB_NAME=jumpserver  # Database to which JumpServer writes data

    # To enable TLS/SSL for external MySQL, see https://docs.jumpserver.org/en/v4/installation/security_setup/mysql_ssl/
    # DB_USE_SSL=true

    ################################## Redis Configuration ###########################
    # Enter valid Redis information for an external Redis service.
    # The system configures a built-in Redis service automatically.
    REDIS_HOST=redis  # Redis address. The value redis starts the built-in Redis container.
    REDIS_PORT=6379  # Redis port
    REDIS_PASSWORD=****************  # Example Redis password

    # To use external Redis Sentinel, configure the following values manually.
    # REDIS_SENTINEL_HOSTS=mymaster/192.168.100.1:26379,192.168.100.1:26380,192.168.100.1:26381
    # REDIS_SENTINEL_PASSWORD=your_sentinel_password
    # REDIS_PASSWORD=your_redis_password
    # REDIS_SENTINEL_SOCKET_TIMEOUT=5

    # To enable TLS/SSL for external Redis, see https://docs.jumpserver.org/en/v4/installation/security_setup/redis_ssl/
    # REDIS_USE_SSL=true

    ################################## Access Configuration ##########################
    # External service port. Change it if it conflicts with an existing service.
    HTTP_PORT=80  # JumpServer web interface port

    ################################## HTTPS Configuration ###########################
    # See https://docs.jumpserver.org/en/v4/installation/proxy/
    # HTTPS_PORT=443
    # SERVER_NAME=your_domain_name
    # SSL_CERTIFICATE=your_cert
    # SSL_CERTIFICATE_KEY=your_cert_key
    #

    # Nginx upload and download size limit.
    CLIENT_MAX_BODY_SIZE=4096m

    ################################ Component Configuration #########################
    # Component registration address. Components register with the Core container by default.
    # In a cluster, change this to the cluster VIP address.
    #
    CORE_HOST=http://core:8080  # JumpServer URL used for API registration requests
    PERIOD_TASK_ENABLED=true

    # Core session settings.
    # SESSION_COOKIE_AGE is the number of idle seconds before a session expires.
    # SESSION_EXPIRE_AT_BROWSER_CLOSE=true expires the session when the browser closes.
    # SESSION_COOKIE_AGE=86400
    SESSION_EXPIRE_AT_BROWSER_CLOSE=true

    # Trusted DOMAINS.
    # Specify trusted access IP addresses. For public access, use the applicable public IP address.
    # DOMAINS="demo.jumpserver.org:443"
    # DOMAINS="172.17.200.191:80"
    # DOMAINS="demo.jumpserver.org:443,172.17.200.191:80"
    DOMAINS="192.168.1.100:80"

    # All components start by default. Set <COMPONENT>_ENABLED to 0 for a component that is not required.
    # CORE_ENABLED=0
    # CELERY_ENABLED=0
    # KOKO_ENABLED=0
    # LION_ENABLED=0
    # MAGNUS_ENABLED=0
    # CHEN_ENABLED=0
    # Enable font smoothing in Lion for a better visual experience.
    JUMPSERVER_ENABLE_FONT_SMOOTHING=true

    ################################## X-Pack Configuration ##########################
    # X-Pack options have no effect in the Community Edition.
    SSH_PORT=2222  # Port used to access JumpServer and assets over SSH
    RDP_PORT=3389  # Port used by Razor to connect to assets over RDP
    XRDP_PORT=3390  # Port used by XRDP to connect to assets over RDP
    MAGNUS_MYSQL_PORT=33061  # Port used by Magnus to connect to MySQL
    MAGNUS_MARIADB_PORT=33062  # Port used by Magnus to connect to MariaDB
    MAGNUS_REDIS_PORT=63790  # Port used by Magnus to connect to Redis
    MAGNUS_POSTGRESQL_PORT=54320  # Port used by Magnus to connect to PostgreSQL
    MAGNUS_SQLSERVER_PORT=14330  # Port used by Magnus to connect to SQL Server
    MAGNUS_ORACLE_PORT=15210  # Port used by Magnus to connect to Oracle
    XRDP_ENABLED=0

    ################################## Other Configuration ###########################
    # The terminal uses the host HOSTNAME as its identifier. This is generated during initial installation.
    SERVER_HOSTNAME=jumpserver-v4

    # Built-in load balancing. If the web interface reports the wrong client IP address, set USE_LB to 0.
    # USE_LB=1 uses: proxy_set_header X-Forwarded-For $remote_addr
    # USE_LB=0 uses: proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
    USE_LB=1

    # Current JumpServer version. Generated automatically after installation and upgrade.
    CURRENT_VERSION={{ jumpserver.tag }}
    ```

> For default and additional parameters, see [Configuration Parameters](env.md).

### 2.2 Modify config.txt

The core JumpServer configuration file is `config.txt`. Its default location is `/opt/jumpserver/config/config.txt`.

After changing `config.txt` while JumpServer is running, restart JumpServer with `jmsctl restart`.

### 2.3 Modify Other Configuration Files

JumpServer must be restarted to load changes made to other configuration files, including:

- Nginx configuration: `/opt/jumpserver/config/nginx/lb_http_server.conf`
- MySQL configuration: `/opt/jumpserver/config/mariadb/mariadb.cnf`
- PostgreSQL configuration: `/data/jumpserver/postgresql/data/postgresql.conf`

After changing any of these files while JumpServer is running, restart JumpServer with `jmsctl restart`.

> Attention: Back up the database before changing database configuration.

### 2.4 Configure and Replace HTTPS Certificates

#### Enable HTTPS for the First Time

1. Upload the certificate files to `/opt/jumpserver/config/nginx/cert/`. This is the fixed default mapped directory and cannot be changed. Certificates are generally named `server.crt` and private keys `server.key`. The filenames must match the values in the configuration file.

2. Stop JumpServer:

    ```bash
    jmsctl stop
    ```

3. Edit the HTTPS section in `/opt/jumpserver/config/config.txt`. Uncomment the parameters and enter the values for the environment:

    ```ini
    HTTPS_PORT=443
    SERVER_NAME=your_domain_name  # Replace with the domain name or IP address used by this environment.
    SSL_CERTIFICATE=server.crt
    SSL_CERTIFICATE_KEY=server.key
    ```

4. Start and verify JumpServer:

    ```bash
    jmsctl start
    # Confirm that port 443 is mapped to the jms_web container.
    docker ps -a
    ```

    Open the JumpServer login page over `https://`. If the browser reports no certificate security warning, the certificate is active.

#### Replace an Expiring Certificate

If the new and old certificate files use the same names, you do not need to change `config.txt`. Replace the certificate without stopping JumpServer:

```bash
# 1. Enter the certificate directory and back up the old certificate.
cd /opt/jumpserver/config/nginx/cert/
mv server.crt server.crt.backup
mv server.key server.key.backup

# 2. Upload the new files and rename them to match the configuration.
mv <new-certificate-file>.crt server.crt
mv <new-private-key-file>.key server.key

# 3. Reload Nginx gracefully inside the Web container.
docker exec -it jms_web nginx -s reload
```

Refresh the browser page and confirm that it displays the new certificate information.

## 3 Database Backup

Back up the JumpServer database regularly to prevent data loss if the JumpServer system fails.

### 3.1 Back Up with the Command-Line Tool

!!! tip ""
    ```bash
    jmsctl backup_db
    ```

### 3.2 Manual Backup Commands

#### Back Up MySQL Manually

> For a built-in database, run the command inside the database container.

!!! tip ""
    ```bash
    mysqldump -u$DB_USER -p$DB_PASSWORD jumpserver > jumpserver-$(date +"%Y-%m-%d").sql
    ```

#### Back Up PostgreSQL Manually

> For a built-in database, run the command inside the database container.

!!! tip ""
    ```bash
    pg_dump -U $DB_USER -h localhost -d jumpserver -f jumpserver-$(date +"%Y-%m-%d").dump
    ```

### 3.3 Scheduled Automatic Backups { #scheduled-automatic-backups }

In production, use a crontab schedule to back up the database automatically and avoid missed manual backups.

```bash
# Edit the current user's crontab.
crontab -e
```

Add the following entries. Adjust the frequency and retention period for your business requirements:

```bash
# Back up the JumpServer database every day at 02:00.
# Use `which jmsctl` to confirm the absolute jmsctl path.
0 2 * * * /usr/local/bin/jmsctl backup_db >> /var/log/jmsctl_backup.log 2>&1

# At 03:00 every day, delete backup files older than 30 days.
# Adjust the directory for the configured VOLUME_DIR.
0 3 * * * find /data/jumpserver/db_backup/ -name "*.sql" -mtime +30 -delete

# Back up the core configuration directory every Sunday at 04:00.
0 4 * * 0 tar -czf /data/jumpserver/db_backup/jumpserver-config-$(date +\%F).tar.gz /opt/jumpserver/config/
```

**Backup strategy recommendations:**

- Backups are stored in `/data/jumpserver/db_backup/` on the local host by default. To avoid losing data and backups in the same server failure, regularly synchronize backups to another host with `scp` or `rsync`, or upload them to object storage.
- Perform a recovery exercise regularly, such as once every quarter, to confirm that the backups are usable.
- Include `/opt/jumpserver/config/`, including `config.txt` and certificates, in the backup scope as well as the database.

## 4 Database Restore

Use the following procedures to roll back the database after a database node failure, failed upgrade, or similar event.

> Attention: You can roll back a database only if a database backup already exists.

### 4.1 Restore a Single-Node Database with the Command-Line Tool

!!! tip ""
    ```bash
    jmsctl restore_db <backup_file_path>
    # The file path must be absolute. Backups are stored in /data/jumpserver/db_backup by default.
    ```

### 4.2 Manual Restore Commands

> Notes:
>
> 1. Manual restore is primarily intended for external databases. For built-in databases, use `jmsctl restore_db` when possible, as described in section 4.1.
> 2. When manually restoring a built-in database, `jmsctl stop` also stops its database container. Start the database container separately first (`docker start jms_postgresql` for built-in PostgreSQL or `docker start jms_mysql` for built-in MySQL), and then run the restore command inside the container.

#### Roll Back a Single-Node MySQL Database

!!! tip ""
    ```bash
    # 1. Stop JumpServer on the JumpServer node to prevent writes during the restore.
    jmsctl stop

    # 2. Restore the database on the database server. For a built-in database, run this inside its container.
    mysql -u$DB_USER -p$DB_PASSWORD jumpserver < /path/to/backup/jumpserver-YYYY-MM-DD.sql

    # 3. Start JumpServer.
    jmsctl start
    ```

#### Roll Back a Single-Node PostgreSQL Database

!!! tip ""
    ```bash
    # 1. Stop JumpServer on the JumpServer node to prevent writes during the restore.
    jmsctl stop

    # 2. Restore the database on the database server. For a built-in database, run this inside its container.
    psql -U $DB_USER -h localhost -d jumpserver -f /path/to/backup/jumpserver-YYYY-MM-DD.dump

    # 3. Start JumpServer.
    jmsctl start
    ```

### 4.3 Restore a Multi-Node Database

!!! info "If your Enterprise Edition environment uses a complex database architecture, contact your customer success team through the enterprise support group for recovery assistance."

Follow the recovery procedure for the database cluster architecture used by your environment.

## 5 Disk Space Management { #disk-space-management }

During long-term operation, session recordings, task logs, database backups, and other files continue to consume disk space. A full disk can prevent database writes, stop recordings from being saved, and cause other failures. Inspect and clean disk space regularly.

### 5.1 Inspect Disk Usage

```bash
# Display disk usage for each mount point.
df -h

# Find the ten largest directories under the persistence directory.
# Adjust the path for the configured VOLUME_DIR.
du -sh /data/jumpserver/* | sort -rh | head -10
```

**Primary sources of disk growth:**

| Directory | Content | Description |
| :--- | :--- | :--- |
| `$VOLUME_DIR/core/data/media/` | Session recordings, uploaded files, downloaded files, and similar content. | Grows with the number of sessions and is usually the largest directory. |
| `$VOLUME_DIR/core/data/celery/` | Asynchronous task logs. | Grows as automation tasks run. |
| `$VOLUME_DIR/core/data/logs/` and each component's `data/logs/` | System and component logs. | Grows much faster when the log level is `DEBUG`. |
| `$VOLUME_DIR/db_backup/` | Database backup files. | Each backup adds a file, so old backups must be removed regularly. |
| `$VOLUME_DIR/postgresql/` | Persistent data for the built-in database. | Grows with business data. **Never delete files directly from this directory.** |

### 5.2 Cleanup Strategy

1. **Configure automatic cleanup:** In the web console, configure retention periods for login logs, operation logs, task logs, session recordings, and other data under the periodic cleanup settings. JumpServer removes expired data automatically.
2. **Remove expired database backups:** Follow the scheduled cleanup example in [3.3 Scheduled Automatic Backups](#scheduled-automatic-backups) and delete old backups according to the retention policy.
3. **Move session recordings:** For long-term retention, configure object storage such as S3 or OSS and move recordings off the local host. See `SERVER_REPLAY_STORAGE` in [Configuration Parameters](env.md).
4. **Configure disk alerts:** Monitor disk usage on each JumpServer node and alert before capacity is exhausted, for example when usage exceeds 80%.

!!! warning "Attention"
    - Before removing session recordings, confirm the organization's audit and compliance retention requirements. Do not remove recordings that must still be retained.
    - Never delete files directly from persistent database directories such as `postgresql/` or `redis/`.

## 6 Routine Inspection

Operations staff should use the following checklist to inspect the JumpServer environment and identify problems early. Adjust the inspection frequency according to the importance of the environment.

| Item | Recommended frequency | Method | Expected result |
| :--- | :--- | :--- | :--- |
| Container status | Daily | `jmsctl status` or `docker ps -a` | All containers are `Up` or `healthy`; no container repeatedly restarts. |
| Disk space | Daily | `df -h` | Usage is below 80%. If it is higher, follow [Disk Space Management](#disk-space-management). |
| Memory and CPU | Daily | `free -h`, `top` | Memory is not exhausted and no component has sustained high load. |
| Database backups | Daily | `ls -lh /data/jumpserver/db_backup/` | Backups are generated on schedule and have reasonable file sizes. |
| Online sessions | Daily | Online sessions page in the web console | No sessions remain open unexpectedly or originate from an unusual source. |
| Core logs | Weekly | Inspect Core logs as described in [Component Log Inspection](logs_inspection.md). | No recurring `ERROR` messages. |
| Web access | Weekly | Open the JumpServer login page in a browser. | The page opens normally and the HTTPS certificate is not close to expiration. |
| Server time | Weekly | `timedatectl` | NTP synchronization is active. MFA code validation depends on accurate system time. |
| Sample asset connections | Weekly | Test connections to a representative set of commonly used assets. | SSH, RDP, and other applicable connection methods work normally. |
