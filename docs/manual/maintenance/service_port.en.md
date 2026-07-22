# Service Ports

## 1 Port Description

JumpServer uses the following ports:

| Port | Purpose | Description |
| :--- | :--- | :--- |
| 22 | SSH | Used for installation, upgrades, and administration. |
| 80 | Web HTTP service | Accesses the JumpServer web interface over HTTP. |
| 443 | Web HTTPS service | Accesses the JumpServer web interface over HTTPS. |
| 3306 | Database service | Used by the MySQL service. |
| 6379 | Database service | Used by the Redis service. |
| 3389 | Razor service port | TCP port required by the RDP proxy service for Windows assets. |
| 2222 | SSH client | Connects to JumpServer using a terminal client such as Xshell, PuTTY, or MobaXterm. |
| 33061 | Magnus MySQL service port | TCP port required by the proxy service for MySQL database assets. |
| 33062 | Magnus MariaDB service port | TCP port required by the proxy service for MariaDB database assets. |
| 54320 | Magnus PostgreSQL service port | TCP port required by the proxy service for PostgreSQL database assets. |
| 63790 | Magnus Redis service port | TCP port required by the proxy service for Redis database assets. |
| 15210 | Magnus Oracle service port | TCP port required by the proxy service for Oracle database assets. |
| 15900 | NEC service port | TCP port required by the proxy service for VNC assets. |

!!! info "Client Access and Proxy Service Ports"
    The client access and proxy service ports in the table (`2222`, `3389`, `33061`, `33062`, `54320`, `63790`, `15210`, and `15900`) allow users to connect to target assets with local client tools such as Windows Remote Desktop Connection (`mstsc`), MobaXterm, Xshell, PuTTY, or a database client.

    When using a local client tool, ensure TCP connectivity from the user PC to the corresponding client access or proxy service port on the JumpServer host. Allow the port through endpoint firewalls and any intermediate network devices. The JumpServer host must also be able to reach the service port used by the target asset.

    `User PC -> JumpServer host client access/proxy service port -> Target asset service port`

## 2 Firewall Configuration

Before deploying JumpServer in production, configure firewalls and other network restrictions to allow the required JumpServer ports.

If you add firewall rules to a running JumpServer node or change its network configuration, restart both Docker and JumpServer:

1. Change the firewall rules or host network configuration.
2. Restart Docker.
3. Restart JumpServer.

!!! tip ""
    ```bash
    # Change the network configuration as required
    # Restart Docker
    systemctl restart docker
    # Restart JumpServer
    jmsctl restart
    ```
