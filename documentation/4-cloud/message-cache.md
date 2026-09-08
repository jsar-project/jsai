# AIUI 消息缓存

`cacheAIUIMessage` 允许服务端为当前认证账号和指定 AIUI 智能体暂存一条消息。适合服务端在用户下一次打开或返回 AIUI 页面时传递路径、自定义数据和文本内容。

接收端通过 AIUI 页面侧的 `api.agent.getAIUICacheMessage()` 原子获取并删除消息。写入接口属于服务端云端集成，不应从 AIUI 页面直接调用。

## 缓存一条消息

```text
POST /metis/openApi/v1/cacheAIUIMessage
Content-Type: application/json
```

```bash
curl --location 'https://<aiui-host>/metis/openApi/v1/cacheAIUIMessage' \
  --header 'Content-Type: application/json' \
  --header "access_token: ${ACCOUNT_ACCESS_TOKEN}" \
  --data '{
    "agentId": "7665068936018264064",
    "path": "/pages/agent/message",
    "customData": "custom-data",
    "content": "您有一条新的智能体消息"
  }'
```

账号由 `access_token` 对应的认证上下文确定。不要在 JSON 请求体中传入 `accountId` 或 `access_token`。

## 请求字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `agentId` | `string` | 是 | 接收消息的 AIUI 智能体 ID |
| `path` | `string` | 是 | 接收后要使用的 AIUI 页面路径 |
| `customData` | `string` | 是 | 应用自定义字符串数据 |
| `content` | `string` | 是 | 消息内容 |

四个字段都必须提供。使用 `path` 跳转页面、解析 `customData` 或展示 `content` 前，应在写入端和接收端按业务约束校验长度与格式。

## 缓存生命周期

- 缓存条目按账号 ID 和 `agentId` 隔离。
- 消息在写入后保存 10 分钟，超时后不可再消费。
- 同一账号、同一 `agentId` 的新消息会覆盖旧消息，不会追加为消息队列。
- `getAIUICacheMessage()` 获取和删除是原子操作；成功消费后，第二次调用不会返回同一条消息。
- 接收端不要盲目重试消费，因为消息可能已经删除，而后续页面处理尚未完成。

## 处理响应

成功响应使用通用结构：

```ts
type CommonV1Response = {
  code: number;
  msg: string;
  timestamp: number;
  uuid: string;
  data: Record<string, unknown>;
};
```

仅当 `code === 1` 时视为缓存成功。其他业务状态结合 `msg` 和 `uuid` 排查；认证信息非法时返回 `401`。日志中不得记录账号 token 或其他认证细节。
