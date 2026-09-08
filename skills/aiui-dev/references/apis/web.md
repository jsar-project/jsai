# AIUI Web Networking and Encoding API Reference

This file documents the currently verified browser-style networking and text decoding APIs available to AIUI agent code.

- Common scope, entry points, and authoring rules live in [apis.md](./index.md).
- Treat this file as the implementation-aligned reference for streamed HTTPS consumption and incremental text decoding.

## `fetch(url, options?)`

### Return behavior

- `fetch(url, options?)` returns a `Promise<Response>`.

### Behavior notes

- Prefer `fetch` when you want promise-based request flow or streamed body consumption through `response.body`.
- If you only care about the final buffered result, `response.text()`, `response.json()`, and `response.arrayBuffer()` remain valid choices.

## `Headers`

### Constructor

- `new Headers(init?)`

### Confirmed behavior

- Current compatibility checks explicitly cover case-insensitive key lookup.
- Repeated header values are merged and returned from `get(name)` as a comma-separated string.
- Verified methods in current examples include `has(name)` and `get(name)`.

### Example

```javascript
const headers = new Headers([
  ['X-Test', 'one'],
  ['x-test', 'two'],
]);

console.log(headers.has('x-test')); // true
console.log(headers.get('X-Test')); // "one, two"
```

## `Response`

### Common properties

- `ok`
- `status`
- `statusText`
- `url`
- `body`
- `bodyUsed`

### Common methods

- `clone()`
- `text()`
- `json()`
- `arrayBuffer()`

### Behavior notes

- `body` is exposed as a `ReadableStream`.
- Once the body is locked by `getReader()`, convenience readers such as `text()` and `json()` no longer consume that same body.
- `bodyUsed` becomes `true` after the body has been consumed or locked to a reader.
- `clone()` allows the original response and the cloned response to be consumed independently.

## `ReadableStream`

### Confirmed usage

- The current documented usage focuses on `response.body.getReader()` for streamed HTTPS responses.
- Use `reader.read()` to pull incremental chunks.
- If `reader.releaseLock` exists and you no longer need the reader, release the lock before reusing the stream object elsewhere.

### Example

```javascript
const response = await fetch('https://example.com/stream');
const reader = response.body.getReader();

while (true) {
  const { value, done } = await reader.read();
  if (done) {
    break;
  }

  console.log('chunk bytes:', value.byteLength);
}
```

## `TextDecoder`

### Constructor

- `new TextDecoder(label?, options?)`

### Common properties

- `encoding`
- `fatal`
- `ignoreBOM`

### Methods

- `decode(input?, options?)`

### Behavior notes

- `decode(input, { stream: true })` keeps the decoder state across chunk boundaries.
- Call `decode()` once more with no input after the last chunk to flush any buffered trailing bytes.
- For streamed UTF-8 text, this is the preferred pattern because multibyte characters may span multiple chunks.

### Example

```javascript
const response = await fetch('https://example.com/stream');
const reader = response.body.getReader();
const decoder = new TextDecoder('utf-8');

let text = '';

while (true) {
  const { value, done } = await reader.read();
  if (done) {
    break;
  }

  text += decoder.decode(value, { stream: true });
}

text += decoder.decode();
console.log(text);
```

## `WebSocket`

Create a connection with `new WebSocket(url, protocols?)`. Important members are `url`, `readyState`, `protocol`, `extensions`, `bufferedAmount`, `binaryType`, `send(data)`, and `close(code?, reason?)`.

Handle `open`, `message`, `error`, and `close` through event listeners or `onopen`, `onmessage`, `onerror`, and `onclose`. A message event exposes `data`; close events expose `code`, `reason`, and `wasClean`.

```javascript
const socket = new WebSocket('wss://example.com/events');
socket.addEventListener('open', () => socket.send('ready'));
socket.addEventListener('message', (event) => console.log(event.data));
socket.addEventListener('close', (event) => {
  console.log(event.code, event.reason, event.wasClean);
});
```

Close the socket when its owning Page, Widget, or task no longer needs it.

## `File` and `FormData`

`new File(parts, name, options?)` creates a Blob with `name`, `lastModified`, `size`, and `type`. `FormData` supports `append()`, `set()`, `get()`, `getAll()`, `has()`, `delete()`, and iteration.

```javascript
const attachment = new File(['AIUI'], 'note.txt', { type: 'text/plain' });
const form = new FormData();
form.append('attachment', attachment);
form.append('category', 'example');
await fetch('/api/upload', { method: 'POST', body: form });
```

Do not manually set a multipart boundary when passing `FormData` as the request body.

## `URL` and `URLSearchParams`

Use `new URL(input, base?)` to parse or resolve URLs. Read or update `protocol`, `host`, `hostname`, `port`, `pathname`, `search`, `hash`, `origin`, and `searchParams`.

`URLSearchParams` supports construction from a query string, records, or pairs plus `append()`, `set()`, `get()`, `getAll()`, `has()`, `delete()`, `sort()`, iteration, and `toString()`.

```javascript
const url = new URL('/weather', 'https://example.com');
url.searchParams.set('city', 'Hangzhou');
console.log(url.href);
```

## Text Encoding

- `new TextEncoder().encode(text)` returns UTF-8 bytes.
- `TextEncoder.encodeInto(source, destination)` writes into an existing buffer.
- `TextDecoder.decode(input?, { stream? })` decodes bytes; use streaming mode across chunk boundaries and finish with an empty `decode()`.

## Web Crypto

### `crypto`

- `crypto.randomUUID(): string`
- `crypto.getRandomValues(typedArray): typedArray`
- `crypto.subtle.digest(algorithm, data): Promise<ArrayBuffer>`
- `crypto.subtle.importKey(format, keyData, algorithm, extractable, keyUsages): Promise<CryptoKey>`
- `crypto.subtle.sign(algorithm, key, data): Promise<ArrayBuffer>`

`getRandomValues()` accepts integer typed arrays and limits one request to 65,536 bytes. Do not use ordinary random-number generators for security tokens.

```javascript
const bytes = new Uint8Array(16);
crypto.getRandomValues(bytes);
const digest = await crypto.subtle.digest(
  'SHA-256',
  new TextEncoder().encode('AIUI'),
);
```

## User Timing

Use `performance.now()` for monotonic elapsed time. User Timing provides marks and measures:

```javascript
performance.mark('render-start');
// Perform the operation.
performance.mark('render-end');
performance.measure('render', 'render-start', 'render-end');

const [entry] = performance.getEntriesByName('render', 'measure');
console.log(entry.duration);
performance.clearMarks();
performance.clearMeasures();
```

## Authoring Rules For Agents

- Prefer `fetch` over `wx.request` when you need `async/await` flow or streamed body consumption.
- Prefer `wx.request` when you need Mini Program-style task callbacks or `RequestTask` control.
- For streamed text, pair `response.body.getReader()` with `TextDecoder.decode(value, { stream: true })` and a final empty `decode()` flush.
- Do not mix `getReader()` with `text()`, `json()`, or `arrayBuffer()` on the same body unless you deliberately use `response.clone()`.
