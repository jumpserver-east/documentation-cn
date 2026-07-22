# 组件日志查看

## 1 系统组件日志

JumpServer 的数据持久化目录（VOLUME_DIR）默认为 `/data/jumpserver`，具体路径可执行以下命令查看：

```bash
cat /opt/jumpserver/config/config.txt | grep VOLUME_DIR
```

本次示例以 `/data/jumpserver/` 为例：

JumpServer 的核心日志存放在 `/data/jumpserver/core/data/logs`。

**核心日志文件详细介绍**

| 日志文件名 | 说明 |
| :--- | :--- |
| `ansible.log` | ansible 执行任务产生的日志（linux 测试资产可连接性、更新硬件信息、推送系统用户、linux 执行改密计划等） |
| `beat.log` | 定时任务的日志 |
| `celery_ansible.log` | 异步任务 ansible 队列下的任务日志 |
| `celery_default.log` | 异步任务默认队列下的任务日志 |
| `celery.log` | celery 组件的日志 |
| `daphne.log` | Django 的一部分，主要用来支持 websocket |
| `drf_exception.log` | 使用 DRF 框架抛出的异常信息 |
| `flower.log` | 作业中心的任务监控组件日志 |
| `gunicorn.log` | 用来记录请求的日志 |
| `jumpserver.log` | JumpServer 的总日志 |
| `unexpected_exception.log` | JumpServer 报错信息日志 |

**其他组件的日志文件位置**

| 组件名称 | 日志文件路径 |
| :--- | :--- |
| Celery | `/data/jumpserver/celery/data/logs` |
| Lion | `/data/jumpserver/lion/data/logs` |
| Koko | `/data/jumpserver/koko/data/logs` |
| Razor| `/data/jumpserver/razor/data/logs` |
| Xrdp | `/data/jumpserver/xrdp/data/logs` |
| Chen | `/data/jumpserver/chen/data/logs` |
| Magnus | `/data/jumpserver/magnus/data/logs` |
| Web | `/data/jumpserver/web/data/logs` |
| Facelive | `/data/jumpserver/facelive/data/logs` |
| Nec | `/data/jumpserver/nec/data/logs` |

> 说明：Web 页面的访问日志（nginx 访问日志）位于 Web 组件的日志目录下，排查页面访问类问题时可结合查看。

## 2 Docker 日志查看

**示例：查看 core 容器的后 100 行日志**

```bash
docker logs -f jms_core --tail 100
```

**查看其他组件的实时日志**

```bash
# 通过容器 ID 查看
docker logs -f [Container ID]

# 通过容器名称查看
docker logs -f [Container name]
```

**按时间范围过滤日志**

```bash
# 查看最近 30 分钟的日志
docker logs --since 30m jms_core

# 查看指定时间点之后的日志
docker logs --since "2026-07-21T10:00:00" jms_core

# 组合使用：查看最近 1 小时内的后 200 行日志
docker logs --since 1h --tail 200 jms_core
```

## 3 调整日志级别

JumpServer 默认日志级别由 `config.txt` 中的 `LOG_LEVEL` 参数控制（可选 `DEBUG`、`INFO`、`WARN`、`ERROR`）。排查问题时可临时调整为 `DEBUG` 以输出更详细的日志信息。

**操作步骤：**

1. 修改 `/opt/jumpserver/config/config.txt` 中的日志级别：

    ```ini
    LOG_LEVEL=DEBUG
    ```

2. 重启 JumpServer 服务使配置生效：

    ```bash
    jmsctl restart
    ```

!!! warning "注意"
    `DEBUG` 级别日志量较大，会明显加快磁盘空间消耗。问题排查结束后，请及时将 `LOG_LEVEL` 调回原级别并重启服务。
