---
name: "aiui-cloud-integration"
description: "Use when implementing server-side AIUI cloud integrations for account token lookup, temporary AIUI message caching, Rokid Glasses notification delivery, or notification-triggered page navigation."
---

# AIUI Cloud Integration

Use this skill for server-side AIUI Cloud calls. Do not use these interfaces from AIUI page code. For page-side `createOpenAPI()` calls such as `getProfile`, `invokeAgentApi`, `saveThirdToken`, `getThirdToken`, or `getAIUICacheMessage`, use the `aiui-cloud-apis` skill instead.

## Choose an integration

| Need | Interface | Authentication |
|---|---|---|
| Get account information for an account token | `GET /account/v1/token` | `access_token` header; official platform agents only |
| Cache one message for an account and AIUI agent | `POST /metis/openApi/v1/cacheAIUIMessage` | `access_token` header |
| Push a notification to a Rokid Glasses user | `CloudIntegration.sendNotification()` or `POST https://rcs.rokid.com/metis/callback/message` | Rokid account SK as Bearer token |

## Keep credentials server-side

Treat the account `access_token` and Rokid account SK as secrets. Read them from environment variables or a secret manager. Never put them in AIUI page code, client bundles, source control, logs, request bodies, or user-visible output.

## Get account information for a token

`getToken` is available only to official platform agents. The current contract returns a string and does not define an internal JSON shape, so do not invent response fields.

```bash
curl --location 'https://<aiui-host>/account/v1/token' \
  --header "access_token: ${ACCOUNT_ACCESS_TOKEN}"
```

- HTTP: `GET /account/v1/token`
- Success: `200 application/json`, with a string response schema.
- Invalid authentication: `401`.
- Do not parse the response as an undocumented object.

## Cache an AIUI message

Use `cacheAIUIMessage` when a server needs to leave one temporary message for a specific AIUI agent. The receiving AIUI app consumes it through page-side `api.agent.getAIUICacheMessage()`.

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

All four body fields are required strings:

| Field | Meaning |
|---|---|
| `agentId` | Target AIUI agent ID |
| `path` | Destination AIUI page path |
| `customData` | Application-defined string data |
| `content` | Message content |

The account comes from authentication; never send `accountId` or `access_token` in the JSON body. Entries are scoped by account ID and `agentId`, expire after 10 minutes, and a new message for the same account and agent overwrites the old message. Treat `code === 1` as success. Validate `path`, `customData`, and `content` before the receiver uses them for routing or display.

## Send a Glasses notification

Install the Node.js 20+ package:

```bash
npm install @yodaos-pkg/cloud-integration
```

```js
import { CloudIntegration } from '@yodaos-pkg/cloud-integration'

const cloud = new CloudIntegration({
  token: process.env.ROKID_SK,
})

const response = await cloud.sendNotification({
  messageId: 'message-unique-id',
  accountId: 'target-account-id',
  message: {
    agentId: 'agent-id',
    content: 'Notification text shown on the Glasses.',
  },
})
```

Constructor options:

- `token` is the required Rokid account SK.
- `endpoint` optionally overrides the default endpoint.
- `fetch` optionally supplies a custom fetch implementation.

When the package is unsuitable, call the endpoint directly. SDK fields are camelCase; HTTP JSON fields are snake_case.

```bash
curl --location 'https://rcs.rokid.com/metis/callback/message' \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer ${ROKID_SK}" \
  --data '{
    "message_id": "message-unique-id",
    "account_id": "target-account-id",
    "message": {
      "agent_id": "agent-id",
      "content": "Notification text shown on the Glasses."
    }
  }'
```

`messageId`, `accountId`, `message.agentId`, and `message.content` must be non-empty strings. Generate a unique `messageId` for tracing and deduplication.

## Open a page from a notification

Add `tool` under `message`. Its `name` must exactly match a route registered in `app.json`. Values in `parameters.properties` pass through unchanged, so the destination page input schema must declare them.

```js
await cloud.sendNotification({
  messageId: 'outfit-message-id',
  accountId: 'target-account-id',
  message: {
    agentId: 'agent-id',
    content: 'View outfit suggestions',
    tool: {
      name: 'pages/cloth/index',
      parameters: {
        type: 'object',
        properties: { field: 'value' },
      },
    },
  },
})
```

When `tool` is present, `tool.name` and `tool.parameters` are required, `tool.parameters.type` must be `"object"`, and `properties` must be an object.

## Success and failures

`sendNotification()` resolves with the complete response only when `code === 1` and `data.success === true`. Invalid input throws `CloudIntegrationValidationError`; network, HTTP, parsing, and server-side business failures throw `CloudIntegrationError`. Use the response `msg` and `uuid` for diagnostics, but never log the SK or Authorization header.

Retry only clearly transient network or server failures, keep the same `messageId` for the same attempted notification, and never retry validation failures or loop indefinitely when delivery status is unknown.

For the package API, also consult [`packages/cloud-integration/README.md`](../../packages/cloud-integration/README.md) when working in this repository.
