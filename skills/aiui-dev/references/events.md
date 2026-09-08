# AIUI Events and Lifecycle Reference

Load this reference when implementing lifecycle, user input, focus, voice wakeup, head gestures, or environment awareness.

## Page Lifecycle

Common Page callbacks are:

| Callback | Use |
| --- | --- |
| `onLoad(query)` | Read route input and initialize state. |
| `onShow()` | Resume work needed while visible. |
| `onReady()` | Access rendered nodes after the initial render. |
| `onHide()` | Pause visible-only timers, listeners, or work. |
| `onUnload()` | Release final Page resources. |

Make repeated show/hide cycles safe. Final cleanup belongs in `onUnload()`.

## Widget Lifecycle

| Callback | Use |
| --- | --- |
| `onCreate()` | Initialize data and one-time resources. |
| `onAttach()` | Refresh visible data and resume visible-only work. |
| `onDetach()` | Pause work that is unnecessary while hidden. |
| `onDestroy()` | Cancel requests, remove listeners, and release resources. |

Do not substitute Page lifecycle callbacks in a Widget.

## Agent Worker Open Event

`onOpen(event)` runs whenever a Page or Widget opens. Call `event.waitUntil(promise)` synchronously for async work whose lifetime must be included in the event.

## Template Events

Use `bind<event>` for normal propagation and `catch<event>` to stop propagation where supported:

```xml
<button bindtap="submit">Submit</button>
<view catchtap="dismiss">...</view>
```

Handlers live on the default-exported logic object. Read payload values from `event.detail`, target information from `event.target`, and declared dataset values from the target dataset.

Do not use DOM `onclick` attributes or assume browser `addEventListener()` behavior for template component events.

## Focus and Default Actions

Interactive elements can receive focus. Style focus visibly and do not remove all focus indication. Some key or navigation events have framework default behavior; prevent it only when the agent intentionally replaces that behavior.

## Key Events

Implement key handlers on the Page logic object using the exact callback names supported by the target. Avoid depending on desktop keyboard codes when the agent targets glasses controls. Keep the UI operable through focus and the device's primary confirmation/back actions.

## Voice Wakeup and World Awareness

Page-level awareness is opt-in:

```javascript
export default {
  onLoad() {
    this.enableWorldAwareness();
  },
  onHeadGesture(event) {
    if (event.gesture === 'nod') this.confirm();
  },
  onOrientationStabilityChange(event) {
    if (event.stable) this.setData({ stable: true });
  },
};
```

`enableWorldAwareness()` is Page-only. Do not call it from Widgets or Agent Workers. Treat wakeup and gesture callbacks as optional signals and preserve another usable interaction path when the product requires it.
