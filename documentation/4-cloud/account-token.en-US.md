# Account Token Lookup

`getToken` lets a server retrieve the account information associated with an account-center token. It is available only to official platform agents and is not part of the app-facing OpenAPI used by AIUI pages.

## Call the interface

```text
GET /account/v1/token
```

Send the account token in the `access_token` header:

```bash
curl --location 'https://<aiui-host>/account/v1/token' \
  --header "access_token: ${ACCOUNT_ACCESS_TOKEN}"
```

`access_token` contains account-center authentication. Although the current contract marks the header as optional, the interface requires valid authentication and returns `401` for invalid credentials. Read the credential from a server-side environment variable or secret manager. Never place it in source code, logs, or responses.

## Handle the response

A successful response has:

- HTTP status: `200`
- Content-Type: `application/json`
- Response schema: string

```text
<token-or-account-information>
```

The current contract does not define the string's internal format. Do not assume it is a JSON object or invent fields such as `accountId` or `userName`. Parse a more detailed structure only after the server contract explicitly defines one.

Invalid authentication returns `401`. Check the credential source, expiry, and calling environment during diagnosis, but never log the token itself.

## Restrictions

- Only official platform agents may call this interface.
- Use it only from a server; do not request it directly from an AIUI page.
- Do not put `access_token` in the query string or request body.
- Do not treat the undefined response string as a structured account object.
