### `api.auth.checkUserAuth()` Check User Information Permission

Purpose: check whether the current app has permission to access sensitive information of the current request user. Use the result only for logging or user-facing guidance; it is not a prerequisite or hard gate for subsequent API calls.

Call without arguments:

```ts
const api = await createOpenAPI();
const auth = await api.auth.checkUserAuth();
```

This method accepts no request arguments. Do not pass `agentId` or an empty object.

Response type:

```ts
type CheckUserAuthResponse = {
  code: number;
  msg: string;
  timestamp: number;
  uuid: string;
  data: {
    checkResult: boolean;
    msg: string;
  };
};
```

Check example:

```ts
if (auth?.data?.checkResult !== true) {
  console.info(auth?.data?.msg || 'User information permission check did not pass');
}

// Continue the business flow. Restricted fields may be unavailable.
const profile = await api.account.getProfile();
if (profile.mobile) {
  useMaskedMobile(profile.mobile);
}
```

Notes:

- `checkResult === true` means the user has signed with the developer and the current app has permission to access the requested sensitive user information.
- If `checkResult === false` or the check throws, log or show an appropriate message, but do not block subsequent API calls solely because of this result.
- A failed or unavailable check may still allow partial information to be returned. Treat restricted fields such as `mobile` and `code` as potentially missing or unavailable and check each field before use.
- Do not expose `uuid`, authentication data, or unnecessary permission details in the UI.
