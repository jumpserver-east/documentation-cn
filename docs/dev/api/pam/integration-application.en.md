## /api/v1/accounts/integration-applications/

### GET
- **Description:**
Get a list of integrated applications, supporting paging, sorting and keyword search.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Query Params:**

| Parameter name | Description | Default value |
| --- | --- | --- |
| limit | Type: Int, number returned per page | - |
| offset | Type: Int, paging offset | - |
| order | Type: String, sort field | - |
| search | Type: String, search keyword | - |

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| count | Type: Int, total number of records |    |
| next | Type: String, next page link | null if no more data |
| previous | Type: String, link to previous page | null if there is no previous page |
| results | Type: List[Object], integrated application list | See table below |

**results[] field description**

| Field name | Field description | Remarks |
| --- | --- | --- |
| id | Type: String(UUID), integrated application ID |    |
| name | Type: String, integrated application name |    |
| logo | Type: String(URL), application logo |    |
| accounts | Type: Object, authorized account information | The return structure is consistent with the backend configuration |
| ip_group | Type: String[], IP range allowed to access | The default value is ["*"] |
| accounts_amount | Type: Int, number of authorized accounts | read-only fields |
| org_id | Type: String, organization ID | read-only fields |
| org_name | Type: String, organization name | read-only fields |
| is_active | Type: boolean, whether to enable |    |
| date_last_used | Type: String(date-time), most recently used time | may be null |
| date_created | Type: String(date-time), creation time | read-only fields |
| date_updated | Type: string(date-time), update time | read-only fields |
| comment | Type: String, remarks |    |

- **Request Example**

**CURL**
```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/?offset=0&limit=15' \
	-H 'Authorization: Bearer b96810faac725563304dada8c323c4fa061863d4' \
	-H 'X-JMS-ORG: 00000000-0000-0000-0000-000000000002'
```

- **Use Case:**

Scenario: During security inspection, search for integrated applications whose name contains jenkins by keyword and confirm whether the relevant CI application is still enabled.

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/?search=jenkins&limit=10&offset=0' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

### POST
- **Description:**
To create an integrated application, you need to specify the name, authorization account and other information.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Request body parameters (Body):**

| Parameter name | Description | Default value |
| --- | --- | --- |
| name* | Type: String, integrated application name | - |
| logo | Type: String(URL), application logo address | - |
| accounts* | Type: Object, authorized account configuration | See backend definition |
| ip_group | Type: String[], IP range allowed to access | ["*"] |
| is_active | Type: boolean, whether to enable | true |
| comment | Type: String, remarks | - |

> Note: Parameters with * are required.

- **Return parameters:**

The returned fields are the same as the `results[]` field description in the GET interface.

- **Request Example**

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

- **Use Case:**

Scenario: Create an exclusive integrated application for the CMDB scheduled synchronization script, authorize only the specified account, and restrict access to the intranet synchronization server network segment.

```sh
curl -X POST 'https://localhost/api/v1/accounts/integration-applications/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>' \
	-H 'Content-Type: application/json' \
	-d '{
		"name": "cmdb-sync",
		"accounts": {"ids": ["1c3f9a2e-5b6d-4e7f-8a9b-0c1d2e3f4a5b"]},
		"ip_group": ["192.168.10.0/24"],
		"is_active": true,
		"comment": "CMDB Special for scheduled synchronization scripts"
	}'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

## /api/v1/accounts/integration-applications/{id}/

### GET
- **Description:**
Get details based on integration app ID.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Path Params:**

| Name | Description | Remarks |
| --- | --- | --- |
| id* | Type: String(UUID), integrated application ID | - |

> Note: Parameters with * are required.

- **Return parameters:**

The returned fields are the same as the `results[]` field description in the GET list interface.

- **Use Case:**

Scenario: When handling the key call exception alarm work order, check the integrated application details according to the application ID, and check the number of authorized accounts and the latest usage time.

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/8f2f6f3e-9d5c-4c1b-b6a7-2f0c9d8e7a61/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

### PUT / PATCH
- **Description:**
To update integrated application information, you can use PUT override or PATCH partial update.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Path Params:**

| Name | Description | Remarks |
| --- | --- | --- |
| id* | Type: String(UUID), integrated application ID | - |

- **Request body parameters (Body):**

The same as creating an interface, PUT needs to provide all required fields, and PATCH only needs to provide the fields that need to be changed.

> Note: Parameters with * are required.

- **Return parameters:**

The returned fields are the same as the `results[]` field description in the GET list interface.

- **Use Case:**

Scenario: It is discovered that the key of an integrated application is suspected to be leaked. First, temporarily disable it through PATCH, block external calls, and then investigate.

```sh
curl -X PATCH 'https://localhost/api/v1/accounts/integration-applications/8f2f6f3e-9d5c-4c1b-b6a7-2f0c9d8e7a61/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>' \
	-H 'Content-Type: application/json' \
	-d '{"is_active": false}'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

### DELETE
- **Description:**
Delete the specified integrated application.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |

- **Path Params:**

| Name | Description | Remarks |
| --- | --- | --- |
| id* | Type: String(UUID), integrated application ID | - |

> Note: Parameters with * are required.

- **Return parameters:**

If the deletion is successful, 204 is returned with no return body.

- **Use Case:**

Scenario: The old monitoring platform has been offline, delete its corresponding integrated applications, and completely reclaim the platform's API access rights.

```sh
curl -X DELETE 'https://localhost/api/v1/accounts/integration-applications/3a7d5c1b-0e9f-4b8a-9c6d-5e4f3a2b1c0d/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

## /api/v1/accounts/integration-applications/{id}/secret/

### GET
- **Description:**
Get the one-time password of the integrated application and only return sensitive content on the first call.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |
| Content-Type | application/json | The request/response body is in JSON format |

- **Path Params:**

| Name | Description | Remarks |
| --- | --- | --- |
| id* | Type: String(UUID), integrated application ID | - |

> Note: Parameters with * are required.

- **Return parameters:**

The returned fields are the same as the `results[]` field description in the GET list interface. Sensitive information is only returned on the first request.

- **Use Case:**

Scenario: After creating a new integrated application cmdb-sync, the administrator calls this interface for the first time to obtain the one-time ciphertext and immediately writes it to the credential management tool of the synchronization server to avoid hard coding in the script.

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/1f6b2c8d-4e5a-4b7c-9d0e-3a2b1c4d5e6f/secret/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

## /api/v1/accounts/integration-applications/account-secret/

### GET
- **Description:**
Obtain desensitized information of accounts associated with integrated applications for auditing or external system synchronization.

- **Request headers:**

| key | value | Remarks |
| --- | --- | --- |
| Authorization | Bearer b96810faac725563304dada8c323c4fa061863d4 | Administrator’s token information. |
| X-JMS-ORG | 00000000-0000-0000-0000-000000000002 | Organization ID. If left blank, it defaults to the Default organization. |

- **Return parameters:**

| Field name | Field description | Remarks |
| --- | --- | --- |
| asset | Type: String, asset name |    |
| asset_id | Type: String(UUID), asset ID | may be empty |
| account | Type: string, account name |    |
| account_id | Type: String(UUID), account ID | may be empty |

- **Use Case:**

Scenario: The security audit platform regularly pulls the desensitization list of accounts associated with each integrated application every week, and checks whether the actual authorization scope is consistent with the change work order.

```sh
curl -X GET 'https://localhost/api/v1/accounts/integration-applications/account-secret/' \
	-H 'Authorization: Bearer <token>' \
	-H 'X-JMS-ORG: <organizationID>'
```

> For the complete integration scenario, please refer to: [Practical case: external script to obtain account password without hard coding](../examples/secret_retrieval.md)

