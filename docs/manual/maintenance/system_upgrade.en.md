# System Upgrade

!!! info "If your Enterprise Edition environment has a complex architecture, contact your customer success team through the enterprise support group for upgrade assistance."

> Attention:</br>
> 1. Before upgrading an environment, review the release notes on the official website to understand recent changes and select an appropriate target version.</br>
> 2. JumpServer services will be unavailable during the upgrade, typically for 10 to 30 minutes depending on the environment. Reserve a maintenance window of at least one hour so there is sufficient time for validation and rollback.

JumpServer provides a one-command upgrade. The process restarts all services on the JumpServer platform and automatically migrates the database schema.

## 1 Upgrade a Single Node

### 1.1 Download and Upload the Installation Package

- Download the latest Enterprise Edition package from the FIT2CLOUD Support Portal, or obtain it from the customer success team in your enterprise support group.

!!! tip ""
    - The Enterprise Edition and Community Edition packages are different. Do not use them interchangeably.
    - Enterprise package format: `jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz` [jumpserver-ee-system-version-system-architecture]

After downloading the package, upload it to the server.

### 1.2 Back Up the Existing Environment

```bash
jmsctl backup_db
```

### 1.3 Extract the Package and Enter the New Package Directory

```bash
tar -xf jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz
cd jumpserver-ee-{{ jumpserver.tag }}-x86_64/
```

### 1.4 Run the Upgrade Script

!!! warning "Run this step from inside the new installation package directory. Verify the current path before continuing."

```bash
./jmsctl.sh upgrade
```

### 1.5 Start JumpServer

```bash
./jmsctl.sh start
```

## 2 Upgrade Multiple Nodes

!!! info "If your Enterprise Edition environment has a complex architecture, contact your customer success team through the enterprise support group for upgrade assistance."

### 2.1 Download and Upload the Installation Package

- Download the latest Enterprise Edition package from the FIT2CLOUD Support Portal, or obtain it from the customer success team in your enterprise support group.

!!! tip ""
    - The Enterprise Edition and Community Edition packages are different. Do not use them interchangeably.
    - Enterprise package format: `jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz` [jumpserver-ee-system-version-system-architecture]

After downloading the package, upload it to every JumpServer node.

### 2.2 Back Up the Existing Environment

```bash
jmsctl backup_db
```

### 2.3 Stop JumpServer on Every Node

```bash
jmsctl stop
```

### 2.4 Extract the Package and Enter the New Package Directory

```bash
tar -xf jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz
cd jumpserver-ee-{{ jumpserver.tag }}-x86_64/
```

### 2.5 Run the Upgrade Script on the Primary Node

Do not upgrade nodes concurrently because simultaneous database schema changes can conflict.

!!! warning "Run this step from inside the new installation package directory. Verify the current path before continuing."

```bash
./jmsctl.sh upgrade
```

### 2.6 Upgrade Each Remaining Node

```bash
./jmsctl.sh upgrade
```

### 2.7 Start the Service on Every Node

```bash
./jmsctl.sh start
```

## 3 Roll Back a Failed Upgrade

!!! warning "Attention"
    - Rollback restores the database backup created before the upgrade. **Users, assets, permissions, sessions, and other data created after the upgrade will be lost.** Proceed carefully.
    - Before deleting a database, confirm that a valid pre-upgrade backup exists.
    - Enterprise Edition customers should confirm the rollback plan with their customer success team through the enterprise support group.

### 3.1 Prerequisites

- A database backup from before the upgrade. This can be the backup created automatically during the upgrade or by `jmsctl backup_db`. Backups are stored in `/data/jumpserver/db_backup/` by default.
- The extracted installation package for the previous version.
- The database address, username, and password.

### 3.2 Stop JumpServer

```bash
jmsctl stop
```

### 3.3 Roll Back the Database

Delete the upgraded database, recreate it, and import the pre-upgrade backup.

**MySQL:**

```bash
# Connect directly to an external database. For a built-in database, enter its container first.
# docker exec -it jms_mysql /bin/bash
mysql -h<database-address> -u<database-user> -p
```

```sql
-- Inspect the database definition and record its character set for the recreated database.
show create database jumpserver;
-- Delete the upgraded database.
drop database jumpserver;
-- Recreate the database with the original character set. MySQL 8.0 generally uses utf8mb3; MySQL 5.7 uses utf8.
CREATE DATABASE `jumpserver` DEFAULT CHARACTER SET utf8mb3;
```

```bash
# Import the pre-upgrade backup.
jmsctl restore_db /data/jumpserver/db_backup/<pre-upgrade-backup>.sql
```

**PostgreSQL:**

```bash
# Connect directly to an external database. For a built-in database, enter its container first.
# docker exec -it jms_postgresql /bin/bash
psql -U <database-user> -h <database-address>
```

```sql
-- Delete and recreate the upgraded database.
DROP DATABASE jumpserver;
CREATE DATABASE jumpserver;
```

```bash
# Import the pre-upgrade backup.
jmsctl restore_db /data/jumpserver/db_backup/<pre-upgrade-backup>.sql
```

### 3.4 Restore the Previous JumpServer Version

```bash
# Enter the extracted package directory for the previous version.
cd <previous-package-path>/
# Reload the previous images when using an offline package.
./jmsctl.sh load_image
# Start the service.
./jmsctl.sh start
```

### 3.5 Validate the Rollback

Log in to the JumpServer web interface. Verify the version, number of users, number of assets, number of permissions, and all critical functions.
