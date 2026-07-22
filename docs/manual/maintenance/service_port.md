# 服务端口说明

## 1 端口说明

JumpServer 服务涉及以下端口：

| 端口 | 作用 | 说明 |
| :--- | :--- | :--- |
| 22 | SSH | 安装、升级及管理使用 |
| 80 | Web HTTP 服务 | 通过 HTTP 协议访问 JumpServer 前端页面 |
| 443 | Web HTTPS 服务 | 通过 HTTPS 协议访问 JumpServer 前端页面 |
| 3306 | 数据库服务 | MySQL 服务使用 |
| 6379 | 数据库服务 | Redis 服务使用 |
| 3389 | Razor 服务端口 | Windows 资产 RDP 代理服务所需的 TCP 端口 |
| 2222 | SSH Client | SSH Client 方式使用终端工具连接 JumpServer，比如 Xshell、PuTTY、MobaXterm 等终端工具 |
| 33061 | Magnus MySQL 服务端口 | MySQL 数据库资产代理服务所需的 TCP 端口 |
| 33062 | Magnus MariaDB 服务端口 | MariaDB 数据库资产代理服务所需的 TCP 端口 |
| 54320 | Magnus PostgreSQL 服务端口 | PostgreSQL 数据库资产代理服务所需的 TCP 端口 |
| 63790 | Magnus Redis 服务端口 | Redis 数据库资产代理服务所需的 TCP 端口 |
| 15210 | Magnus Oracle 服务端口 | Oracle 数据库资产代理服务所需的 TCP 端口 |
| 15900 | NEC 服务端口 | VNC 资产代理服务所需的 TCP 端口 |

!!! info "客户端接入与代理服务端口"
    表中的客户端接入与代理服务端口（`2222`、`3389`、`33061`、`33062`、`54320`、`63790`、`15210`、`15900`）用于用户通过本地客户端工具连接目标资产，例如 Windows 远程桌面连接（`mstsc`）、MobaXterm、Xshell、PuTTY 或数据库客户端。

    使用本地客户端工具时，必须确保用户 PC 到 JumpServer 宿主机对应客户端接入或代理服务端口的 TCP 网络可达，并在两端防火墙及中间网络设备上放行相应端口。JumpServer 宿主机还必须能够访问目标资产实际使用的服务端口。

    `用户 PC -> JumpServer 宿主机客户端接入/代理服务端口 -> 目标资产服务端口`

## 2 防火墙配置说明

在生产环境部署时，如有防火墙以及网络限制应提前开放 JumpServer 相应的访问端口。

当 JumpServer 环境运行中需要添加 JumpServer 节点服务器防火墙策略以及修改机器网络配置等，需要重启 docker 服务以及 JumpServer 服务。操作步骤如下：
1. 修改防火墙策略或修改机器网络配置；
2. 重启 docker 服务；
3. 重启 JumpServer 服务。

!!! tip ""
    ```bash
    # 修改网络相关配置（根据实际需求执行）
    # 重启 docker 服务
    systemctl restart docker
    # 重启 JumpServer 服务
    jmsctl restart
    ```
