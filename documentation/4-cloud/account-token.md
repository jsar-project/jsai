# 账号 Token 查询

`getToken` 用于在服务端根据账号中心 token 获取对应的账号信息。该接口仅供平台官方智能体使用，不属于 AIUI 页面侧 OpenAPI。

## 调用接口

```text
GET /account/v1/token
```

调用时在请求头中携带 `access_token`：

```bash
curl --location 'https://<aiui-host>/account/v1/token' \
  --header "access_token: ${ACCOUNT_ACCESS_TOKEN}"
```

`access_token` 是账号中心认证信息。虽然当前协议将请求头标记为可选，但接口依赖有效认证，认证信息非法时返回 `401`。调用方应在可信服务端从环境变量或密钥管理服务中读取凭据，不要将其写入源码、日志或响应。

## 处理响应

成功时返回：

- HTTP 状态：`200`
- Content-Type：`application/json`
- 响应 schema：字符串

```text
<token-or-account-information>
```

当前协议没有定义字符串的内部格式，因此不要假设它是某个 JSON 对象，也不要臆造 `accountId`、`userName` 等响应字段。只有在服务端协议进一步明确后，才能按更细结构解析。

认证失败时返回 `401`。排查时检查 token 来源、有效期和调用环境，但不要把 token 本身记录到日志。

## 使用限制

- 仅平台官方智能体可以调用。
- 只能在服务端使用，不要从 AIUI 页面直接请求。
- 不要将 `access_token` 放入查询参数或请求体。
- 不要把未定义格式的字符串作为结构化账号对象处理。
