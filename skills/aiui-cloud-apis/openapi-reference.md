# AIUI Cloud OpenAPI Reference

This is the shared protocol reference for the six AIUI app-facing Rokid OpenAPI operations. `SKILL.md` is the short routing entry; this file owns the cross-operation rules and links to complete per-operation references.

## Scope and operation map

The agent-facing operations are `getProfile`, `invokeAgentApi`, `checkUserAuth`, `saveThirdToken`, `getThirdToken`, and `getAIUICacheMessage`. Server-side `cacheAIUIMessage`, `getToken`, and `sendNotification` are documented separately under `pr_resource/cloud_integration_docs/`.

| Namespace | Operation | HTTP path | Request body | Detailed reference |
|---|---|---|---|---|
| `api.account` | `getProfile` | `GET /account/v1/profile` | None | [get-profile](references/get-profile.md) |
| `api.agent` | `invokeAgentApi` | `POST /metis/openApi/v1/invokeAgentApi` | Required | [invoke-agent-api](references/invoke-agent-api.md) |
| `api.auth` | `checkUserAuth` | `POST /metis/openApi/v1/checkUserAuth` | None | [check-user-auth](references/check-user-auth.md) |
| `api.agent` | `saveThirdToken` | `POST /metis/openApi/v1/saveThirdToken` | Required | [save-third-token](references/save-third-token.md) |
| `api.agent` | `getThirdToken` | `POST /metis/openApi/v1/getThirdToken` | None | [get-third-token](references/get-third-token.md) |
| `api.agent` | `getAIUICacheMessage` | `POST /metis/openApi/v1/getAIUICacheMessage` | None | [get-aiui-cache-message](references/get-aiui-cache-message.md) |

## Retrieval keywords

OpenAPI, Rokid OpenAPI, AIUI OpenAPI, Ink OpenAPI, createOpenAPI, import open, requestBody, body wrapper, getProfile, invokeAgentApi, checkUserAuth, saveThirdToken, getThirdToken, getAIUICacheMessage, third-party token, save third-party token, Lingzhu agent, SSE, text/event-stream, user information permission, one-time message consumption, access_token, x-request-agent-id, headIcon, userName, accountId, mobile, AgentOutput, final_answer, metadata, 获取账号信息, 灵珠智能体, 用户信息权限检查, 保存第三方Token, 获取第三方Token, 获取缓存消息.

## General Rules

- Access OpenAPI only through the built-in Ink runtime module `'open'`:
  ```ts
  import { createOpenAPI } from 'open';
  ```
- `'open'` is a QuickJS/Ink built-in module, not an npm package.
- Always `await createOpenAPI()` before calling any OpenAPI method.
- Prefer calling OpenAPI inside page lifecycle methods or page methods. Do not wrap these calls as global methods in `app.js`.
- Always guard dynamic import, `createOpenAPI()`, and OpenAPI calls with `try/catch`.
- In normal AIUI app code, do not manually pass `access_token`; the runtime host injects authentication headers.
- Do not call `fetch('/openapi/...')` directly unless the user explicitly asks for low-level HTTP debugging.
- Do not import `createOpenAPI` from `@yodaos-pkg/ink`.
- Do not assume `api` is a global variable.
- If an OpenAPI operation defines a `requestBody`, the runtime method argument must contain `body`: `api.method({ body: requestBody })`. Omitting `body` causes the runtime error `is missing required request body`.
- Treat the `body` wrapper as required even when constructing the request dynamically. Put all request-body fields inside `body`; do not flatten them into the outer method arguments.
- Operations without a `requestBody` are called without a `body` wrapper or placeholder object unless they define other parameters.

## Recommended Page Pattern

```ts
Page({
  data: {
    profile: null,
    loading: true,
    error: ''
  },

  async onLoad() {
    await this.loadProfile();
  },

  async loadProfile() {
    let api = null;
    try {
      const { createOpenAPI } = await import('open');
      api = await createOpenAPI();
    } catch {
      this.setData({
        profile: null,
        loading: false,
        error: 'OpenAPI 不可用'
      });
      return;
    }

    try {
      const profile = await api.account.getProfile();
      this.setData({
        profile,
        loading: false,
        error: ''
      });
    } catch {
      this.setData({
        profile: null,
        loading: false,
        error: '无法获取用户信息'
      });
    }
  }
});
```

## Do

- Use `import { createOpenAPI } from 'open'` or `await import('open')`.
- Call `await createOpenAPI()` inside page code.
- Use `try/catch` for OpenAPI unavailable cases, authentication failures, network errors, and runtime errors.
- Null-check fields such as `headIcon`, `metadata`, `content`, and `final_answer`.
- Check `checkUserAuth` with an explicit boolean condition on `data.checkResult`.
- Treat `getThirdToken` as successful when `code === 1`; `data === null` is a no-token empty state, not a failure.
- Keep a returned third-party token confidential and use it only for its intended platform request.
- Call `api.agent.saveThirdToken` with the `{ body: requestBody }` wrapper; omit `accountId` because the API derives it from the logged-in user.
- Treat `getAIUICacheMessage` as successful when `code === 1`; `data === null` is an empty cache state, not a failure.
- Consume a retrieved cache message once; do not retry blindly because successful retrieval deletes it atomically.
- Handle agent SSE responses according to `event`, `step`, `status`, and `message`.

## Don't

- Do not use `@yodaos-pkg/ink`.
- Do not write `api.openapi.account.v1.getProfile()`.
- Do not call `fetch('/openapi/...')` directly.
- Do not manually read, concatenate, pass, or display `access_token`.
- Do not invent undocumented OpenAPI response fields.
- Do not `await api.agent.invokeAgentApi(...)`; it returns an `EventSource` immediately.
- Do not treat the `EventSource` as an ordinary JSON object or final text string; receive events through `result.onmessage`, parse `e.data`, and concatenate non-empty `message.content` fragments.
