### `api.agent.getAIUICacheMessage()` Get and Delete an AIUI Cached Message

Purpose: atomically retrieve and delete the cached message for the current account. Use it for one-time message consumption.

Call without arguments:

```ts
const api = await createOpenAPI();
const response = await api.agent.getAIUICacheMessage();

if (response.code === 1 && response.data) {
  const { path, customData, content } = response.data;
  // Consume the message once.
}
```

This method has no `requestBody` and accepts no request arguments. Do not pass `agentId`, `body`, or an empty object.

Response type:

```ts
type AIUICacheMessageData = {
  agentId: string;
  accountId: string;
  path: string;
  customData: string;
  content: string;
};

type GetAIUICacheMessageV1Response = {
  code: number;
  msg: string;
  timestamp: number;
  uuid: string;
  data: AIUICacheMessageData;
};
```

Notes:

- The schema declares `data` as `AIUICacheMessageData`, but the endpoint description says no cached message is represented by `data === null`; retain a runtime empty-state check.
- Retrieval and deletion are atomic. Do not expect the same message to be returned by a second call.
- Use the returned `path`, `customData`, and `content` for page routing or display only after validating them for the intended UI flow.
- Do not use returned `accountId` as caller-supplied authorization data; account scoping is enforced by the runtime-authenticated request.
