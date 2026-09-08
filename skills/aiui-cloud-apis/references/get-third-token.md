### `api.agent.getThirdToken()` Get a Third-Party Token

Purpose: get the third-party platform token saved for the current account under the AIUI agent making the request.

Call without arguments:

```ts
const api = await createOpenAPI();
const response = await api.agent.getThirdToken();

if (response.code === 1 && response.data) {
  const { token, refreshToken, meta } = response.data;
  // Use the token values and custom metadata only for the intended third-party request.
}
```

Response type:

```ts
type GetThirdTokenV1Response = {
  code: number;
  msg: string;
  timestamp: number;
  uuid: string;
  data: {
    token: string;
    refreshToken: string;
    meta: string;
  } | null;
};
```

Usage notes:

- This operation has no `requestBody`. Call `api.agent.getThirdToken()` without `body` or an empty placeholder object.
- The runtime supplies the current user's `access_token` and the current AIUI agent ID (`x-request-agent-id`) from the authenticated execution context. Do not manually add these headers or expose their values in application code.
- `code === 1` with `data === null` is a successful response meaning no third-party token is saved for the current account and agent. Handle it as an empty state, not an API error.
- When `data` is non-null, `data.token` is the third-party token, `data.refreshToken` is the third-party refresh token, and `data.meta` is the third-party custom parameter string.
- Treat non-empty `data.token` and `data.refreshToken` values as sensitive credentials. Do not log, display, persist, cache, or return them to the user. Use them only for the intended third-party platform request. Treat `data.meta` as third-party-provided data: validate it before use and avoid exposing it unless explicitly required.
