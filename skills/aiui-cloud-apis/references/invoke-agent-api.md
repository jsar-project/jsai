### `api.agent.invokeAgentApi({ body })` Invoke Lingzhu Agent API

Purpose: call a Lingzhu platform agent from an AIUI app and receive agent text, thought-process events, tool-call events, or error events. The OpenAPI response media type is `text/event-stream`. The method returns an `EventSource` immediately, so do not use `await`; receive events through `result.onmessage`.

Basic call:

```ts
const api = await createOpenAPI();
const result = api.agent.invokeAgentApi({
  body: {
    conversation_id: '1',
    agent_id: 'c1571b19a6c04dcab9b83bd06865eb3i',
    question: '你好'
  }
});

result.onmessage = (e) => {
  console.info('event message', e);
};
```

Call with an image URL:

```ts
const api = await createOpenAPI();
const result = api.agent.invokeAgentApi({
  body: {
    conversation_id: '1',
    agent_id: 'c1571b19a6c04dcab9b83bd06865eb3i',
    question: '请描述这张图片',
    imageUrl: 'https://example.com/image.png'
  }
});

result.onmessage = (e) => {
  console.info('event message', e);
};
```

Request type:

```ts
type AgentApiV1Request = {
  conversation_id?: string;
  agent_id: string;
  question: string;
  imageUrl?: string;
  image?: string;
  imageType?: string;
};
```

Parameter notes:

- `agent_id` and `question` are required.
- `conversation_id` may be supplied by the user. If the user does not supply one, generate a random ID when the AIUI agent is opened.
- Reuse the same `conversation_id` for every request while that AIUI agent session remains open, so the agent retains the conversation context.
- When the user closes and later reopens the AIUI agent, generate a new `conversation_id`. Do not generate a new ID for every message within one open session.
- Prefer `imageUrl` for image input.
- If `imageUrl` is unavailable, pass base64 image content through `image`.
- When passing base64 image content, also pass `imageType`.
- `imageType` defaults to `image/webp`; common values include `image/png` and `image/jpeg`.

SSE `data` payload shape:

```ts
type AgentOutput = {
  step: 'thought' | 'action' | 'content';
  serial_number: number;
  agent_id: string;
  message_id: string | null;
  status: 'tool_calling' | 'tool_call_finsh' | 'streaming' | 'finish';
  message: AgentMessage;
};

type AgentMessage = {
  type: 'text' | 'tool' | 'skill';
  content?: string | null;
  final_answer?: string | null;
  metadata?: Record<string, unknown> | null;
};
```

EventSource event notes:

- Assign `result.onmessage` to receive each event.
- Each callback argument contains `data`, `lastEventId`, and `type`. Parse the `AgentOutput` JSON string with `JSON.parse(e.data)`.
- `e.type === 'data'` / `e.type === 'inProgress'`: usually intermediate streaming output.
- `e.type === 'end'` / `e.type === 'done'`: usually request completion.
- `e.type === 'error'`: a catchable agent execution error that needs to be shown to the user. Parse `e.data` and display `message.content` as the user-facing error message.
- `step: thought` is the thought process. Do not append it to the final user-facing answer by default.
- `step: content` is generated text content. Concatenate every non-empty `message.content` fragment in event order.
- Currently use the concatenated `message.content` as the response; do not depend on or replace it with `message.final_answer`.
- `step: action` usually represents a tool/action call; tool details are in `message.metadata`.
- The OpenAPI schema currently spells the tool completion status as `tool_call_finsh`, while runtime events may return `tool_call_finish`; handle both values defensively.

EventSource message examples:

```text
{ data: '{"step":"content","serial_number":15,"agent_id":"7566562107311572576","message_id":null,"status":"streaming","message":{"type":"text","content":"。","final_answer":null,"metadata":null}}', lastEventId: '15ba39a8b5794051921e4f785c97e7ed', type: 'data' }
{ data: '{"step":"content","serial_number":16,"agent_id":"7566562107311572576","message_id":null,"status":"finish","message":{"type":"text","content":null,"final_answer":"你的手机号是135****0000。","metadata":null}}', lastEventId: '15ba39a8b5794051921e4f785c97e7ed', type: 'end' }
```
EventSource error message example:

```
{ data: '{"step":"content","serial_number":1,"agent_id":"7566562107311572576","message_id":"3728e58480634b97a0b233781cdfedeb","status":"streaming","message":{"type":"text","content":"调用失败","metadata":null}}', lastEventId: '9a57f55d3e2745c28453fed20acf17c2', type: 'error' }
```

Some catchable errors that must be communicated to the user are returned as `error` events. For these events, parse `e.data` and show the non-empty `payload.message.content` to the user instead of silently logging or appending it as normal response content.

Content accumulation example:

```ts
let content = '';

result.onmessage = (e) => {
  console.info('event message', e);

  const payload = JSON.parse(e.data);
  if (payload.step === 'content' && payload.message?.content) {
    content += payload.message.content;
  }

  if (e.type === 'end') {
    console.info('agent response', content);
  }
};
```

Do not append `null` or empty `message.content` values.

Raw SSE text response example:

```text
event:data
data:{"step":"content","serial_number":2,"agent_id":"928536bcf83340c48a0af9a0eec95f36","message_id":"3728e58480634b97a0b233781cdfedeb","status":"streaming","message":{"type":"text","content":"文本响应","final_answer":null,"metadata":null}}

event:end
data:{"step":"content","serial_number":3,"agent_id":"928536bcf83340c48a0af9a0eec95f36","message_id":"3728e58480634b97a0b233781cdfedeb","status":"finish","message":{"type":"text","content":";","final_answer":"文本响应;","metadata":null}}
```

For a text response, consume each non-empty `message.content` from `step: content` events and concatenate the fragments in event order. Currently do not use `message.final_answer`.

Tool response example:

```text
event:end
data:{"step":"action","serial_number":1,"agent_id":"f75f295c11334683b595afad6abf16b2","message_id":"e901877f398341538984ba2609a79d0d","status":"tool_call_finish","message":{"type":"tool","content":null,"final_answer":null,"metadata":{"name":"take_photo","output":{"status":1,"data":{"handling_required":true,"command":"take_photo","params":{"is_recall":true,"action":null,"sensor_mode":0,"name":null,"poi_name":null,"navi_type":null,"title":null,"start_time":null,"end_time":null,"prepay_id":null,"order_title":null,"total_amount":null,"pay_sign":null,"agent_id":null,"model_vendor":null,"tools":null,"agent_name":null,"agent_logo":null,"need_param":null,"prologue":null}}},"prologue":null}}}

event:end
data:{"step":"action","serial_number":1,"agent_id":"f75f295c11334683b595afad6abf16b2","message_id":"e901877f398341538984ba2609a79d0d","status":"finish","message":{"type":"text","content":null,"final_answer":"","metadata":null}}
```

For a tool response, read the tool name and output from `message.metadata`. For example, `message.metadata.name === 'take_photo'` and `message.metadata.output.data.command === 'take_photo'`. A following `finish` event may contain an empty `final_answer`; do not treat it as the tool result.

Thought-process response example:

```text
id:d3a20d4e55a348fba70f8f4a9e0974f6
event:data
data:{"step":"thought","serial_number":1,"agent_id":"928536bcf83340c48a0af9a0eec95f36","message_id":"3728e58480634b97a0b233781cdfedeb","status":"streaming","message":{"type":"text","content":"正在思考","final_answer":null,"metadata":null}}
```

Thought-process content is carried by `step: thought` events and is not included in `final_answer`. Do not concatenate thought content into the final user-facing answer.
