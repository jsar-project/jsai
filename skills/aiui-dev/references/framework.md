# AIUI Framework Reference

Use this reference for project structure, registration, Pages, Widgets, Agent Workers, SFC files, and agent configuration.

## Project Structure

```text
agent/
├── AGENTS.md
├── app.json
├── app.js
├── pages/
│   └── index/index.ink
├── widgets/
│   └── weather/index.ink
├── workers/
│   └── sync.js
├── components/
└── assets/
```

- `AGENTS.md` describes identity and requested capabilities.
- `app.json` declares Pages, Widgets, Agent Workers, and window configuration.
- `app.js` default-exports agent lifecycle and global data.
- Paths in configuration omit the `.ink` extension where shown.

## Registration

All logic entry files use an ES module default export:

```javascript
export default {
  data: { ready: false },
  onLoad() {
    this.setData({ ready: true });
  },
};
```

Never generate `App({...})`, `Page({...})`, or `Widget({...})`.

## Page Declaration

Declare Page routes in order; the first is the default Page:

```json
{
  "pages": ["pages/index/index", "pages/detail/index"]
}
```

A Page supports either of these equivalent source structures:

- **Multi-file Page**: four files with the same base path and the extensions `.json`, `.wxml`, `.wxss`, and `.js`.
- **Single-file Page**: one `.ink` file containing the same configuration, template, style, and logic concerns.

Do not mix the two forms for the same route.

The top-level Page schema describes render-time input:

```json
{
  "description": "Displays weather for a city.",
  "schema": {
    "data": {
      "type": "object",
      "properties": {
        "city": { "type": "string" },
        "temperature": { "type": "number" }
      },
      "required": ["city", "temperature"]
    }
  }
}
```

Keep `description` observable and define every required input in `schema.data`.

## Multi-file Page

For the route `pages/weather/index`, create all four files together:

```text
pages/weather/
├── index.json
├── index.wxml
├── index.wxss
└── index.js
```

`index.json` contains Page configuration and render-input metadata:

```json
{
  "navigationBarTitleText": "Weather",
  "description": "Displays weather for a city.",
  "schema": {
    "data": {
      "type": "object",
      "properties": {
        "city": { "type": "string" },
        "temperature": { "type": "number" }
      },
      "required": ["city", "temperature"]
    }
  }
}
```

`index.wxml` contains the Page template. It does not need an outer `<page>` block:

```xml
<view class="weather">
  <text>{{city}}</text>
  <text>{{temperature}}°C</text>
</view>
```

`index.wxss` contains Page-local styles:

```css
.weather {
  display: flex;
  flex-direction: column;
  padding: var(--spacing-md);
}
```

`index.js` default-exports Page state, lifecycle, and handlers:

```javascript
export default {
  data: {
    city: '',
    temperature: 0,
  },
  onLoad(query) {
    this.setData({ city: query.city || this.data.city });
  },
};
```

The four filenames must share the same basename. The corresponding `app.json` route remains extensionless: `"pages/weather/index"`.

## Single-file `.ink`

```html
<script def>
{
  "navigationBarTitleText": "Weather"
}
</script>

<script setup>
export default {
  data: { city: 'Hangzhou' },
};
</script>

<page>
  <view class="page"><text>{{city}}</text></view>
</page>

<style>
.page { display: flex; }
</style>
```

- `<script def>` contains JSON configuration, not executable JavaScript.
- `<script setup>` contains one default-exported logic object.
- `<page>` or `<widget>` is the sole interface root.
- `<style>` contains page-local WXSS/CSS.

## Widget Development

AIUI supports two Widget families on Rokid Glasses:

| Family | Occupied grid cells | Current Glasses size |
| --- | ---: | ---: |
| `1x1` | 1 | `239 × 140` |
| `1x2` | 2 adjacent cells | `480 × 140` |

Rokid Glasses provide four Widget grid cells. `family` is a size category, not a fixed canvas contract. Use Flexbox, percentages, available space, padding, and content-driven sizing. Never hardcode a Widget root to `239px × 140px`.

Declare the Widget in `app.json`:

```json
{
  "widgets": [
    { "path": "widgets/weather/index", "family": "1x2" }
  ]
}
```

The same family must appear in the Widget file:

```html
<script def>
{ "widget": { "family": "1x2" } }
</script>

<script setup>
export default {
  data: { temperature: 24 },
  refresh() {
    this.setData({ temperature: this.data.temperature + 1 });
  },
};
</script>

<widget>
  <view class="widget" bindtap="refresh">
    <text>{{temperature}}°C</text>
  </view>
</widget>

<style>
.widget {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
```

Widget callbacks are `onCreate()`, `onAttach()`, `onDetach()`, and `onDestroy()`. Attach and detach may repeat. Widgets do not use Page lifecycle callbacks or Page-only `enableWorldAwareness()` and `finish()`.

Open a declared Widget with:

```javascript
window.open('widgets/weather?city=hangzhou', '_widget');
```

This API currently opens Widgets only, not other Pages.

## Agent Worker Development

Agent Workers are non-visual scripts for temporary shared state, one-time work, or a long-lived capability such as a Bluetooth GATT Server. They are not Web Workers.

```json
{
  "agentWorkers": [{
    "name": "sync",
    "script": "workers/sync.js",
    "trigger": { "type": "open" },
    "lifetime": "foreground"
  }]
}
```

| Field | Contract |
| --- | --- |
| `name` | Required, non-empty, and unique in the agent. |
| `script` | Required project-relative `.js` or `.ts`; no URL, absolute path, `..`, or backslash. |
| `trigger` | Required; currently only `{ "type": "open" }`. Only one open-trigger Worker is allowed. |
| `lifetime` | Required: `instant` or `foreground`. `background` is reserved and invalid. |
| `capabilities` | Optional. Currently supports `bluetooth-peripheral`, only with `foreground`. |

Opening a Page or Widget invokes `onOpen(event)`. Register async work before the callback returns:

```javascript
export default {
  refreshPromise: null,
  onOpen(event) {
    event.waitUntil(this.ensureRefreshed());
  },
  ensureRefreshed() {
    if (!this.refreshPromise) {
      this.refreshPromise = this.refresh().finally(() => {
        this.refreshPromise = null;
      });
    }
    return this.refreshPromise;
  },
  async refresh() {
    this.latestStatus = 'ready';
  },
};
```

Making `onOpen()` async does not automatically extend the event. A running Worker retains values on `this`; values reset when it stops. Persist durable state in storage.

Workers provide timers, Promise, `console`, `performance`, URL/text encoding, Web Crypto, WebAssembly, and project ES modules. They do not provide Page/Widget objects, `window`, `document`, `fetch`, routing, rendering, or media capture.

## Modules, Components, and Resources

- Declare custom components in `usingComponents`; use absolute project paths for shared components.
- Project-local ES module resolution is consistent across agent, Page, and Agent Worker entry points.
- Keep packaged images, fonts, audio, and data under project paths such as `/assets/icon.png`.
- Prefer source paths and the project packaging workflow; do not assume direct access to arbitrary device files.

## TypeScript and WebAssembly

Use project-supported `.ts` entry points where declared. Keep public values crossing JavaScript boundaries serializable. For WebAssembly, load packaged modules through the supported module/resource workflow rather than browser-only filesystem assumptions.
