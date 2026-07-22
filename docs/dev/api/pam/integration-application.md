## /api/v1/accounts/integration-applications/

### GET
- **描述：**
获取集成应用列表，支持分页、排序和关键词搜索。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **查询参数（Query Params）：**

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| limit | 类型：Int，每页返回数量 | - |
| offset | 类型：Int，分页偏移量 | - |
| order | 类型：String，排序字段 | - |
| search | 类型：String，搜索关键词 | - |

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| count | 类型：Int，总记录数 |  |
| next | 类型：String，下页链接 | 无更多数据时为 null |
| previous | 类型：String，上页链接 | 无上一页时为 null |
| results | 类型：List[Object]，集成应用列表 | 见下表 |

**results[] 字段说明**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| id | 类型：String(UUID)，集成应用 ID |  |
| name | 类型：String，集成应用名称 |  |
| logo | 类型：String(URL)，应用 Logo |  |
| accounts | 类型：Object，授权账号信息 | 返回结构与后端配置一致 |
| ip_group | 类型：String[]，允许访问的 IP 范围 | 默认值为 ["*"] |
| accounts_amount | 类型：Int，授权账号数量 | 只读字段 |
| org_id | 类型：String，组织 ID | 只读字段 |
| org_name | 类型：String，组织名称 | 只读字段 |
| is_active | 类型：Boolean，是否启用 |  |
| date_last_used | 类型：String(date-time)，最近使用时间 | 可能为 null |
| date_created | 类型：String(date-time)，创建时间 | 只读字段 |
| date_updated | 类型：String(date-time)，更新时间 | 只读字段 |
| comment | 类型：String，备注 |  |

- **请求示例**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/?offset=0&limit=15' \
	-H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
	-H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

- **使用案例：**

场景：安全巡检时，按关键词检索名称包含 jenkins 的集成应用，确认相关 CI 应用是否仍处于启用状态。

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/?search=jenkins&limit=10&offset=0' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

### POST
- **描述：**
创建集成应用，需指定名称、授权账号等信息。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **请求体参数（Body）：**

| 参数名 | 描述 | 默认值 |
| --- | --- | --- |
| name* | 类型：String，集成应用名称 | - |
| logo | 类型：String(URL)，应用 Logo 地址 | - |
| accounts* | 类型：Object，授权账号配置 | 请参阅后端定义 |
| ip_group | 类型：String[]，允许访问的 IP 范围 | ["*"] |
| is_active | 类型：Boolean，是否启用 | true |
| comment | 类型：String，备注说明 | - |

> 注：带 * 的参数为必填项。

- **返回参数：**

返回字段同 GET 接口中 `results[]` 字段说明。

- **请求示例**

**CURL**
```sh
curl -X POST 'https://localhost/api/v1/accounts/integration-applications/' \
	-H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
	-H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002' \
	-H 'Content-Type: application/json' \
	-d '{
		"name": "auto-sync",
		"accounts": {"ids": ["account-id-1"]},
		"ip_group": ["*"]
	}'
```

- **使用案例：**

场景：为 CMDB 定时同步脚本创建专属集成应用，仅授权指定账号，并限制只能从内网同步服务器网段访问。

```sh
curl -X POST 'https://localhost/api/v1/accounts/integration-applications/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>' \
	-H 'Content-Type: application/json' \
	-d '{
		"name": "cmdb-sync",
		"accounts": {"ids": ["1c3f9a2e-5b6d-4e7f-8a9b-0c1d2e3f4a5b"]},
		"ip_group": ["192.168.10.0/24"],
		"is_active": true,
		"comment": "CMDB 定时同步脚本专用"
	}'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

## /api/v1/accounts/integration-applications/{id}/

### GET
- **描述：**
根据集成应用 ID 获取详情。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **路径参数（Path Params）：**

| 名称 | 描述 | 备注 |
| --- | --- | --- |
| id* | 类型：String(UUID)，集成应用 ID | - |

> 注：带 * 的参数为必填项。

- **返回参数：**

返回字段同 GET 列表接口中 `results[]` 字段说明。

- **使用案例：**

场景：处理密钥调用异常告警工单时，根据应用 ID 查看该集成应用详情，核对授权账号数量与最近使用时间。

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/8f2f6f3e-9d5c-4c1b-b6a7-2f0c9d8e7a61/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

### PUT / PATCH
- **描述：**
更新集成应用信息，可使用 PUT 覆盖或 PATCH 局部更新。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **路径参数（Path Params）：**

| 名称 | 描述 | 备注 |
| --- | --- | --- |
| id* | 类型：String(UUID)，集成应用 ID | - |

- **请求体参数（Body）：**

与创建接口相同，PUT 需提供全部必填字段，PATCH 仅需提供需变更字段。

> 注：带 * 的参数为必填项。

- **返回参数：**

返回字段同 GET 列表接口中 `results[]` 字段说明。

- **使用案例：**

场景：发现某集成应用的密钥疑似外泄，先通过 PATCH 将其临时停用，阻断外部调用后再排查。

```sh
curl -X PATCH 'https://localhost/api/v1/accounts/integration-applications/8f2f6f3e-9d5c-4c1b-b6a7-2f0c9d8e7a61/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>' \
	-H 'Content-Type: application/json' \
	-d '{"is_active": false}'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

### DELETE
- **描述：**
删除指定集成应用。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |

- **路径参数（Path Params）：**

| 名称 | 描述 | 备注 |
| --- | --- | --- |
| id* | 类型：String(UUID)，集成应用 ID | - |

> 注：带 * 的参数为必填项。

- **返回参数：**

删除成功返回 204，无返回体。

- **使用案例：**

场景：老旧监控平台已下线，删除其对应的集成应用，彻底回收该平台的 API 访问权限。

```sh
curl -X DELETE 'https://localhost/api/v1/accounts/integration-applications/3a7d5c1b-0e9f-4b8a-9c6d-5e4f3a2b1c0d/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

## /api/v1/accounts/integration-applications/{id}/secret/

### GET
- **描述：**
获取集成应用一次性密文，仅在首次调用时返回敏感内容。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |
| Content-Type | application/json | 请求/响应体为 JSON 格式 |

- **路径参数（Path Params）：**

| 名称 | 描述 | 备注 |
| --- | --- | --- |
| id* | 类型：String(UUID)，集成应用 ID | - |

> 注：带 * 的参数为必填项。

- **返回参数：**

返回字段同 GET 列表接口中 `results[]` 字段说明，敏感信息仅在首次请求时返回。

- **使用案例：**

场景：新建集成应用 cmdb-sync 后，管理员首次调用本接口获取一次性密文，并立即将其写入同步服务器的凭据管理工具，避免在脚本中硬编码。

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/1f6b2c8d-4e5a-4b7c-9d0e-3a2b1c4d5e6f/secret/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

## /api/v1/accounts/integration-applications/account-secret/

### GET
- **描述：**
获取集成应用关联账号的脱敏信息，用于审计或外部系统同步。

- **请求头（Headers）：**

| 键 | 值 | 备注 |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | 管理员的 token 信息。 |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | 组织 ID，留空则默认为 Default 组织 |

- **返回参数：**

| 字段名称 | 字段描述 | 备注 |
| --- | --- | --- |
| asset | 类型：String，资产名称 |  |
| asset_id | 类型：String(UUID)，资产 ID | 可能为空 |
| account | 类型：String，账号名称 |  |
| account_id | 类型：String(UUID)，账号 ID | 可能为空 |

- **使用案例：**

场景：安全审计平台每周定时拉取各集成应用关联账号的脱敏清单，与变更工单核对实际授权范围是否一致。

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/account-secret/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <组织ID>'
```

> 完整集成场景可参考：[实战案例：外部脚本免硬编码获取账号密码](../examples/secret_retrieval.md)

