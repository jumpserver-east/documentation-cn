# 系统升级操作

!!! info "企业版客户如果架构复杂，建议在企业客户支持群中联系客户成功团队获取升级帮助。"

> 注意：</br>
> 1. 如您的环境需要升级，建议首先查看官方网站的 release note，了解近期版本变化，选择适合当前环境的版本。</br>
> 2. JumpServer 服务升级过程中服务会停止一段时间（根据环境情况，时间为 10~30 分钟），建议至少申请一个小时的变更窗口，留足验证以及回退时间。


JumpServer 升级服务采用一键快速升级方式，此过程会重启整个 JumpServer 平台的所有服务，并自动变更数据库表结构；

## 1 单节点升级步骤

### 1.1 下载并上传安装包

- 企业版安装包在飞致云 support 门户中下载最新安装包或在企业支持群中联系相关客户成功团队获取。

!!! tip ""
    - 注意：企业版安装包与社区版安装包不同，请勿混用。
    - 企业版安装包格式：`jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz`  [jumpserver-ee-系统版本-系统架构]

下载完成后，需要上传安装包至服务器后台。

### 1.2 备份原环境服务数据

```bash
jmsctl backup_db
```

### 1.3 解压安装包并进入新版本安装包目录

```bash
tar -xf jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz
cd jumpserver-ee-{{ jumpserver.tag }}-x86_64/
```

### 1.4 执行升级脚本
!!! warning "此步骤需要进入新版本安装包内执行，请注意路径。"

```bash
./jmsctl.sh upgrade
```

### 1.5 启动 JumpServer 服务

```bash
./jmsctl.sh start
```

## 2 多节点升级步骤

!!! info "企业版客户如果架构复杂，建议在企业客户支持群中联系客户成功团队获取升级帮助。"

### 2.1 下载并上传安装包

- 企业版安装包在飞致云 support 门户中下载最新安装包或在企业支持群中联系相关客户成功团队获取。

!!! tip ""
    - 注意：企业版安装包与社区版安装包不同，请勿混用。
    - 企业版安装包格式：`jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz`  [jumpserver-ee-系统版本-系统架构]

下载完成后，需要上传安装包至各个节点服务器后台。

### 2.2 备份原环境服务数据

```bash
jmsctl backup_db
```

### 2.3 停止各个节点的 JumpServer 服务

```bash
jmsctl stop 
```

### 2.4 解压安装包并进入新版本安装包目录

```bash
tar -xf jumpserver-ee-{{ jumpserver.tag }}-x86_64.tar.gz
cd jumpserver-ee-{{ jumpserver.tag }}-x86_64/
```

### 2.5 主节点执行升级脚本（不能同时升级，以免数据库表结构变更冲突）
!!! warning "此步骤需要进入新版本安装包内执行，请注意路径。"

```bash
./jmsctl.sh upgrade
```

### 2.6 其余节点分别进行升级

```bash
./jmsctl.sh upgrade
```

### 2.7 启动各个节点服务

```bash
./jmsctl.sh start
```

## 3 升级失败回滚

!!! warning "注意"
    - 版本回滚是根据升级前备份的数据库文件进行回退的，**升级后新增的用户、资产、授权、会话等数据将无法保留**，请谨慎操作。
    - 执行删除数据库操作前，必须确认升级前的备份文件存在且可用。
    - 企业版客户建议先在企业客户支持群中联系客户成功团队确认回滚方案。

### 3.1 前提条件

- 具备升级前的数据库备份文件（升级时自动备份的文件与 `jmsctl backup_db` 生成的备份文件，默认位于 `/data/jumpserver/db_backup/` 目录下）；
- 保留有升级前旧版本的安装包（解压目录）；
- 已知数据库的连接信息（地址、用户名、密码）。

### 3.2 停止 JumpServer 服务

```bash
jmsctl stop
```

### 3.3 回退数据库

先删除升级后的数据库，再重建数据库并导入升级前的备份文件。

**MySQL 场景：**

```bash
# 连接数据库（外置数据库直接连接；内置数据库先进入容器）
# docker exec -it jms_mysql /bin/bash
mysql -h<数据库地址> -u<数据库用户> -p
```

```sql
-- 查看建库语句，确认字符集（重建数据库时需与原库保持一致）
show create database jumpserver;
-- 删除升级后的数据库
drop database jumpserver;
-- 重建数据库（字符集以上一步查询结果为准，MySQL 8.0 一般为 utf8mb3，MySQL 5.7 为 utf8）
CREATE DATABASE `jumpserver` DEFAULT CHARACTER SET utf8mb3;
```

```bash
# 导入升级前的备份文件
jmsctl restore_db /data/jumpserver/db_backup/<升级前的备份文件>.sql
```

**PostgreSQL 场景：**

```bash
# 连接数据库（外置数据库直接连接；内置数据库先进入容器）
# docker exec -it jms_postgresql /bin/bash
psql -U <数据库用户> -h <数据库地址>
```

```sql
-- 删除升级后的数据库并重建
DROP DATABASE jumpserver;
CREATE DATABASE jumpserver;
```

```bash
# 导入升级前的备份文件
jmsctl restore_db /data/jumpserver/db_backup/<升级前的备份文件>.sql
```

### 3.4 使用旧版本安装包回退服务

```bash
# 进入升级前旧版本的安装包解压目录
cd <旧版本安装包解压路径>/
# 重新加载旧版本镜像（离线安装包场景）
./jmsctl.sh load_image
# 启动服务
./jmsctl.sh start
```

### 3.5 验证回滚结果

登录 JumpServer Web 页面，检查版本信息、用户数量、资产数量、授权数量及各功能是否正常。