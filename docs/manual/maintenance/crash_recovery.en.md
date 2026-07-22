# Fault Recovery

!!! info "Enterprise Edition customers should contact the customer success team through the enterprise support group for prompt assistance with service failures."

## 1 JumpServer Application Failure

If a component container has unusually high load or an `unhealthy` status, use one of the following methods for temporary recovery.

**Method 1: Restart the affected container**

```bash
# Restart the affected container, or use `jmsctl restart [core/celery/koko/chen/lion]` to recreate it.
docker restart <container-name>
# Check the status of all components.
jmsctl status
```

**Method 2: Restart all services**

```bash
# Enter the extracted installation package directory. Use the actual path for your environment.
cd <package-path>/jumpserver-ee-{{ jumpserver.tag }}-x86_64
# Stop and remove all JumpServer services.
./jmsctl.sh down
# Check for containers that did not stop.
docker ps -a
# If any remain, force-stop and remove them. Replace ID with the actual container ID.
docker kill ID
docker rm ID
# Start JumpServer. If an individual component does not start, wait briefly and run the command again.
./jmsctl.sh start
# Check the JumpServer status.
./jmsctl.sh status
```

## 2 Database Failure { #database-failure }

1. Back up the database:

    ```bash
    jmsctl backup_db
    ```

2. Stop JumpServer on the JumpServer node to prevent conflicting writes:

    ```bash
    jmsctl stop
    ```

3. Log in to the database server. Identify and correct the failure, such as a stopped service, invalid configuration, or insufficient disk space.
4. After correcting the failure, start the database service and then start JumpServer:

    ```bash
    jmsctl start
    ```

## 3 Server Failure

1. Restore the server hardware or operating system and confirm that the server is operating normally.
2. Check the built-in or external database. Restore it by following [Database Failure](#database-failure).
3. Start the dependent services:

    ```bash
    # Start Keepalived if the environment uses it.
    systemctl start keepalived
    # Start JumpServer.
    jmsctl start
    ```

4. Run `jmsctl status` to check the service status.

## 4 Common Troubleshooting

### 4.1 Insufficient Disk Space

**Symptoms:** Services behave abnormally, database writes fail, session recordings cannot be saved, or pages report errors.

**Procedure:**

1. Run `df -h` to check disk utilization. Run `du -sh /data/jumpserver/* | sort -rh | head -10` to identify the largest directories.
2. Follow the disk space management section in [Basic Operations](daily_operations.md) to remove expired recordings, task logs, and old backups.
3. After cleanup, run `jmsctl status` to check every component. Run `docker restart <container-name>` for any component in an abnormal state.

### 4.2 Web Interface Is Unavailable or Returns 502

**Symptoms:** The browser cannot open the JumpServer login page, or the page returns a 502 error.

**Procedure:**

1. Run `docker ps -a` and confirm that the `jms_web` and `jms_core` containers are running.
2. If `jms_core` is `starting`, wait for startup to finish and try again.
3. Inspect the Core container logs to identify the error:

    ```bash
    docker logs -f jms_core --tail 100
    ```

4. Confirm that the server is listening on ports 80 and 443 and that the firewall and security group allow the required port.

### 4.3 Component Status Is Unhealthy

**Symptoms:** A component container is shown as `unhealthy` in `jmsctl status` or `docker ps`.

**Common causes and solutions:**

1. **Component registration failed:** Confirm that `BOOTSTRAP_TOKEN` in the component configuration matches Core. A mismatch can occur after migration or reinstallation.
2. **Core is unreachable:** Confirm that `CORE_HOST` is correct and that Core is running normally.
3. Inspect the component logs in `$VOLUME_DIR/<component-name>/data/logs/`. See [Component Log Inspection](logs_inspection.md).
4. After correcting the cause, run `docker restart <container-name>` and use `jmsctl status` to confirm recovery.

### 4.4 MFA Code Validation Fails

**Symptoms:** The user's MFA code is always rejected.

**Common causes and solutions:**

1. Time-based MFA codes require accurate system time. A significant difference between the server and phone clocks causes validation to fail.
2. Run `timedatectl` to verify the JumpServer server time, time zone, and NTP synchronization status.
3. Correct the server time, keep NTP synchronization enabled, and ask the user to obtain a new code and log in again.

### 4.5 User Cannot See Authorized Assets

**Symptoms:** The user can log in to JumpServer, but the asset list is empty or omits some assets.

**Common causes and solutions:**

1. Check whether the applicable asset permission rule has expired.
2. Confirm that the user or user group and the assets are all included in the rule.
3. After changing the rule, ask the user to refresh the page or log in again.

## 5 Security Recommendations

> 1. Back up the JumpServer database regularly to prevent service interruption from data loss. Set a daily or weekly schedule according to business requirements.
> 2. Back up the core JumpServer configuration, including `config.txt` and component configuration files, so the environment can be recovered quickly.
> 3. Enable MFA globally to add a second authentication factor and reduce the risk caused by compromised passwords.
> 4. Enable an account backup plan and regularly send critical account information to a designated email address for emergency recovery.
