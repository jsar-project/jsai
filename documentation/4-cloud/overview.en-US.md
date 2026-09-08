# Cloud Overview

AIUI Cloud provides server-side integration capabilities that connect third-party systems with Rokid cloud services and Glasses devices. Business services, scheduled jobs, and external agents can query account information, temporarily cache AIUI messages, or send notifications to a user's Glasses.

These capabilities must run in a trusted server environment. They are separate from the app-facing OpenAPI that AIUI pages call through the built-in `open` module. Never put an account `access_token`, Rokid account SK, or third-party token in page code, client bundles, source control, or logs.

## Choose a capability

| Need | Interface | Documentation |
|---|---|---|
| Get account information for an account token | `GET /account/v1/token` | [Account token lookup](./account-token.en-US.md) |
| Cache one message for an account and AIUI agent | `POST /metis/openApi/v1/cacheAIUIMessage` | [AIUI message cache](./message-cache.en-US.md) |
| Send a notification or page-navigation action to a user's Glasses | `CloudIntegration.sendNotification()` or the cloud HTTP endpoint | [Sending notifications](./notifications.en-US.md) |

`getToken` is available only to official platform agents. A message written by `cacheAIUIMessage` is atomically retrieved and deleted by the AIUI page-side `api.agent.getAIUICacheMessage()` call. Notification delivery supports both a Node.js SDK and direct HTTP.

## Install the npm package

For notification delivery, prefer `@yodaos-pkg/cloud-integration`. It requires Node.js 20 or later and uses the runtime's built-in `fetch`:

```bash
npm install @yodaos-pkg/cloud-integration
```

```js
import { CloudIntegration } from '@yodaos-pkg/cloud-integration'

const cloud = new CloudIntegration({
  token: process.env.ROKID_SK,
})
```

The SK is a sensitive credential for the account that owns the agent. Store it in a server-side environment variable or secret manager.

## Install the Cloud Integration Skill

The `aiui-cloud-integration` Skill gives AI coding assistants server-side context for account lookup, message caching, notification delivery, page navigation, and error handling:

```bash
npx skills add https://github.com/jsar-project/AIUI/tree/main/skills/aiui-cloud-integration
```

The Skill assists development and code generation. It does not replace credentials or call cloud interfaces automatically.
