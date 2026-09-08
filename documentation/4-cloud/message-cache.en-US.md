# AIUI Message Cache

`cacheAIUIMessage` lets a server store one temporary message for the authenticated account and a specific AIUI agent. Use it to pass a page path, application-defined data, and text that the user can receive the next time they open or return to an AIUI page.

The receiving app atomically retrieves and deletes the message through page-side `api.agent.getAIUICacheMessage()`. The write interface is a server-side cloud integration and must not be called directly from an AIUI page.

## Cache a message

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
    "content": "You have a new agent message"
  }'
```

The authentication context associated with `access_token` determines the account. Do not include `accountId` or `access_token` in the JSON body.

## Request fields

| Field | Type | Required | Description |
|---|---|---|---|
| `agentId` | `string` | Yes | AIUI agent that receives the message |
| `path` | `string` | Yes | AIUI page path used by the receiver |
| `customData` | `string` | Yes | Application-defined string data |
| `content` | `string` | Yes | Message content |

All four fields are required. Before routing with `path`, parsing `customData`, or displaying `content`, validate their length and format against the intended business flow on both the writer and receiver.

## Cache lifecycle

- Entries are scoped by account ID and `agentId`.
- A message remains available for 10 minutes after it is written.
- A new message for the same account and `agentId` overwrites the old message; this is not a message queue.
- `getAIUICacheMessage()` retrieves and deletes atomically. A second call does not return the consumed message.
- The receiver must not retry consumption blindly: the message may already be deleted even if later page processing has not completed.

## Handle the response

A successful call uses the common response shape:

```ts
type CommonV1Response = {
  code: number;
  msg: string;
  timestamp: number;
  uuid: string;
  data: Record<string, unknown>;
};
```

Treat the cache write as successful only when `code === 1`. Use `msg` and `uuid` to diagnose other business results. Invalid authentication returns `401`. Never log the account token or other authentication details.
