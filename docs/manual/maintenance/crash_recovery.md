# 故障恢复操作

!!! info "企业版客户出现服务故障建议在企业客户支持群中联系客户成功团队获取及时帮助。"

## 1 JumpServer 程序故障

当出现组件容器异常高负载或者当容器健康为 unhealthy 时可以通过以下方案临时恢复。

**方案一：重启故障容器**

```bash
# 重启故障容器 或者通过 `jmsctl restart [core/celery/koko/chen/lion]` 重建容器。
docker restart <容器名>
# 检查所有组件运行状态
jmsctl status 

```

**方案二：全部重启**

```bash
# 进入安装包解压目录（以实际解压路径为准）
cd <安装包解压路径>/jumpserver-ee-{{ jumpserver.tag }}-x86_64
# 停止 JumpServer 所有服务
./jmsctl.sh down
# 检查是否有未停止的容器
docker ps -a
# 若有未停止的容器，执行强制停止并删除（替换 ID 为实际容器 ID）
docker kill ID
docker rm ID
# 启动 JumpServer 服务（若个别组件启动不成功，可稍等后再次执行启动命令）
./jmsctl.sh start
# 检查 JumpServer 启动状态
./jmsctl.sh status
```

## 2 数据库故障

1. 备份数据库：

 ```bash
 jmsctl backup_db
 ```

2. 在 JumpServer 节点停止 JumpServer 服务（避免数据写入冲突）：

 ```bash
 jmsctl stop
 ```

3. 登录数据库服务器，检查数据库具体故障原因（如服务未启动、配置错误、磁盘空间不足等），针对性修复；
4. 修复完成后，启动数据库服务，再重启 JumpServer：

 ```bash
 jmsctl start
 ```

## 3 服务器宕机

1. 优先恢复服务器硬件/系统，确保服务器能正常运行；
2. 检查数据库状态（内置/外置），按照 [数据库故障](#2-数据库故障) 的处理方式恢复数据库；
3. 启动相关依赖服务：
 ```bash
 # 若使用 Keepalived，启动 Keepalived 服务
 systemctl start keepalived
 # 启动 JumpServer 服务
 jmsctl start
 ```
4. 通过 `jmsctl status` 检查服务状态。

## 4 常见故障排查

### 4.1 磁盘空间不足

**现象**：服务运行异常、数据库写入失败、会话录像无法保存、页面报错等。

**处理步骤：**

1. 执行 `df -h` 确认磁盘使用率，执行 `du -sh /data/jumpserver/* | sort -rh | head -10` 定位占用空间最大的目录；
2. 参考《日常基本操作》中的磁盘空间管理章节清理过期录像、任务日志与旧备份文件；
3. 清理完成后，通过 `jmsctl status` 检查各组件状态，对状态异常的容器执行 `docker restart <容器名>`。

### 4.2 Web 页面无法访问或报 502

**现象**：浏览器无法打开 JumpServer 登录页，或页面返回 502 错误。

**处理步骤：**

1. 执行 `docker ps -a` 检查 `jms_web`、`jms_core` 容器是否处于运行状态；
2. 若 `jms_core` 正在启动中（starting），等待其完成启动后重试；
3. 查看 core 容器日志定位报错原因：

    ```bash
    docker logs -f jms_core --tail 100
    ```

4. 检查服务器 80/443 端口监听状态以及防火墙、安全组策略是否放行相应端口。

### 4.3 组件状态为 unhealthy

**现象**：`jmsctl status` 或 `docker ps` 中某组件容器健康状态显示 unhealthy。

**常见原因与处理：**

1. **组件注册失败**：检查该组件配置中的 `BOOTSTRAP_TOKEN` 是否与 core 保持一致（迁移或重装后易出现不一致）；
2. **无法访问 core 服务**：检查 `CORE_HOST` 配置是否正确、core 服务是否正常运行；
3. 查看该组件日志定位具体报错（日志位于 `$VOLUME_DIR/<组件名称>/data/logs/` 目录，参考《日志查看操作》）；
4. 处理后执行 `docker restart <容器名>` 重启该组件，并通过 `jmsctl status` 确认恢复。

### 4.4 MFA 验证码校验失败

**现象**：用户输入的 MFA 动态验证码始终提示错误。

**常见原因与处理：**

1. MFA 动态验证码依赖准确的系统时间，服务器时间与手机时间偏差过大会导致校验失败；
2. 执行 `timedatectl` 检查 JumpServer 服务器时间与时区是否正确、NTP 同步是否开启；
3. 校准服务器时间并保持 NTP 同步后，让用户重新获取验证码登录。

### 4.5 用户登录后看不到授权资产

**现象**：用户可以正常登录 JumpServer，但资产列表为空或缺少部分资产。

**常见原因与处理：**

1. 检查对应的资产授权规则是否已过期；
2. 检查用户（或其所在用户组）与资产是否均在授权规则的范围内；
3. 调整授权规则后，让用户刷新页面或重新登录确认。

## 5 安全建议

> 1. 定期备份 JumpServer 数据库，避免数据丢失导致业务中断，建议结合业务场景设置每日/每周备份频率。
> 2. 定期备份 JumpServer 核心配置文件（如 `config.txt` 及各类组件配置文件），便于故障时快速恢复环境。
> 3. 全局开启 MFA（多因素认证）功能，通过二次验证提升账户安全性，降低因密码泄露引发的安全风险。
> 4. 开启账户备份计划，定期将关键账户信息备份至指定邮箱，作为账户异常时的应急逃生方案。
