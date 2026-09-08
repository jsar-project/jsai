# AIUI Device API Reference

This file documents the verified device and sensor APIs available to AIUI agent code.

- Common scope, entry points, and authoring rules live in [apis.md](./index.md).
- Interactive-gate behavior and host capability limits are part of the current implementation surface.
- Do not assume browser-complete Bluetooth or Generic Sensor semantics beyond what is documented here.
- Page-scoped environment awareness callbacks such as `onHeadGesture(event)` live on the page object and are summarized in [SKILL.md](../events.md).

## `navigator.bluetooth`

### Properties

- `navigator.bluetooth`

### Methods

- `getAvailability()`
- `getDevices()`
- `requestDevice(options?)`
- `scanDevices(options?)`

### Return behavior

- `getAvailability()` returns a `Promise<boolean>`.
- `getDevices()` returns a `Promise<BluetoothDevice[]>`.
- `requestDevice(options?)` returns a `Promise<BluetoothDevice>`.
- `scanDevices(options?)` returns a `Promise<BluetoothScan>`.

### Request options

- `acceptAllDevices?: boolean`
- `optionalServices?: string[]`
- `filters?: Array<{ name?: string; services?: string[] }>`

### Behavior notes

- `scanDevices(options?)` is Ink-specific and is not part of the standard Web Bluetooth API.
- `acceptAllDevices` defaults to `false`.
- `optionalServices` is forwarded as a list of service UUID strings.
- `filters` supports only `name` and `services`.
- Empty or partially specified options are normalized before being sent to the native runtime.
- `requestDevice()`, `scanDevices()`, `device.gatt.connect()`, and `characteristic.startNotifications()` require the owning InkView to remain interactive.
- `getAvailability()` and `getDevices()` remain available when the owning InkView is non-interactive.

## `BluetoothScan`

### Methods

- `onDeviceFound(callback)`
- `offDeviceFound(callback?)`
- `stop()`

### Event behavior

- `onDeviceFound(callback)` registers a `devicefound` listener.
- `offDeviceFound(callback?)` unregisters the same listener, or all listeners when omitted.
- `devicefound` listeners receive a `DeviceFoundEvent`.
- `DeviceFoundEvent` extends `Event`.
- `DeviceFoundEvent` exposes `device`.

### Behavior notes

- `BluetoothScan` is backed by the DOM-style event target implementation.
- `stop()` stops the native scan and shuts down the JavaScript event dispatcher.
- `stop()` is synchronous at the JavaScript boundary and performs the native stop asynchronously.

## `BluetoothDevice`

### Properties

- `id`
- `name`
- `gatt`

### Behavior notes

- `id` is a string.
- `name` is a string or `null`.
- `gatt` always returns a `BluetoothRemoteGATTServer` wrapper for the device.

## `BluetoothRemoteGATTServer`

### Properties

- `connected`

### Methods

- `connect()`
- `disconnect()`
- `getPrimaryService(uuid)`
- `getPrimaryServices(uuid?)`

### Return behavior

- `connect()` returns a `Promise<BluetoothRemoteGATTServer>`.
- `disconnect()` returns a `Promise<void>`.
- `getPrimaryService(uuid)` returns a `Promise<BluetoothRemoteGATTService>`.
- `getPrimaryServices(uuid?)` returns a `Promise<BluetoothRemoteGATTService[]>`.

### Behavior notes

- `connected` reflects the current connection state cached by the runtime wrapper.
- `connect()` resolves immediately when already connected.
- `disconnect()` resolves immediately when already disconnected.
- `disconnect()` updates the cached `connected` state after the async disconnect completes.

### Error behavior

- `getPrimaryService(uuid)` throws synchronously when the server is not connected.
- `getPrimaryServices(uuid?)` throws synchronously when the server is not connected.
- Empty service UUID values are rejected before the native call.
- `connect()` fails when the owning InkView is non-interactive.

## `BluetoothRemoteGATTService`

### Properties

- `uuid`
- `isPrimary`

### Methods

- `getCharacteristic(uuid)`
- `getCharacteristics(uuid?)`

### Return behavior

- `getCharacteristic(uuid)` returns a `Promise<BluetoothRemoteGATTCharacteristic>`.
- `getCharacteristics(uuid?)` returns a `Promise<BluetoothRemoteGATTCharacteristic[]>`.

### Behavior notes

- `uuid` is a string.
- `isPrimary` is a boolean.
- `getCharacteristic(uuid)` should be called with a characteristic UUID string.

## `BluetoothRemoteGATTCharacteristic`

### Properties

- `uuid`
- `properties`
- `value`

### Methods

- `readValue()`
- `writeValue(value)`
- `writeValueWithResponse(value)`
- `writeValueWithoutResponse(value)`
- `startNotifications()`
- `stopNotifications()`
- `addEventListener('characteristicvaluechanged', listener)`
- `removeEventListener('characteristicvaluechanged', listener?)`
- `dispatchEvent(event)`

### Return behavior

- `readValue()` returns a `Promise<number[]>`.
- `writeValue(value)` returns a `Promise<void>`.
- `writeValueWithResponse(value)` returns a `Promise<void>`.
- `writeValueWithoutResponse(value)` returns a `Promise<void>`.
- `startNotifications()` returns a `Promise<BluetoothRemoteGATTCharacteristic>`.
- `stopNotifications()` returns a `Promise<BluetoothRemoteGATTCharacteristic>`.

### Behavior notes

- `uuid` is a string.
- `properties` is a `BluetoothCharacteristicProperties`.
- `value` is a cached byte array value or `null`.
- `readValue()` updates the cached `value`.
- When a native notification arrives, the cached `value` is updated and a `characteristicvaluechanged` event is dispatched.
- `startNotifications()` keeps a dedicated event target registration per characteristic instance.

### `BluetoothCharacteristicProperties`

Readonly boolean properties:

- `broadcast`
- `read`
- `writeWithoutResponse`
- `write`
- `notify`
- `indicate`
- `authenticatedSignedWrites`

## `Accelerometer`

### Constructor

- `new Accelerometer(options?)`

### Constructor options

- `frequency?: number`

### Properties

- `x`
- `y`
- `z`
- `timestamp`
- `activated`
- `hasReading`

### Methods

- `start()`
- `stop()`

### Event behavior

- `Accelerometer` inherits from `EventTarget`.
- Supported event names are `activate`, `reading`, and `error`.
- `activate` exposes `sessionId`.
- `reading` exposes `sessionId`, `x`, `y`, `z`, and `timestamp`.
- `error` exposes `sessionId`, `error`, and `message`.

### Behavior notes

- `frequency` is forwarded to the host as a best-effort hint.
- A fresh instance starts with `activated === false` and `hasReading === false`.
- `x`, `y`, `z`, and `timestamp` stay `null` until the first successful reading arrives.
- The first successful reading flips `activated` to `true` and `hasReading` to `true`.
- `stop()` sets `activated` back to `false` but keeps the last successful reading cached.
- `stop()` is a no-op when the instance is already idle.

## `navigator.bluetoothPeripheral`

This optional API is available only inside an Agent Worker that declares `"capabilities": ["bluetooth-peripheral"]`. The Worker must use `lifetime: "foreground"`. It lets nearby BLE devices discover and interact with services provided by the agent.

### `openGattServer(options)`

Returns `Promise<BluetoothLocalGATTServer>`. `options.services` declares the complete service and characteristic structure. Each characteristic has a UUID and properties such as `read`, `write`, and `notify`.

```javascript
const serviceUuid = '12345678-1234-5678-1234-56789abcdef0';
const valueUuid = '12345678-1234-5678-1234-56789abcdef1';

const server = await navigator.bluetoothPeripheral.openGattServer({
  services: [{
    uuid: serviceUuid,
    characteristics: [{
      uuid: valueUuid,
      properties: { read: true, write: true, notify: true },
    }],
  }],
});

const value = server.getService(serviceUuid).getCharacteristic(valueUuid);
value.addEventListener('readrequest', (event) => {
  event.respondWith(new Uint8Array([1]));
});
value.addEventListener('writerequest', (event) => {
  console.log(event.value);
  event.respondWith();
});

await server.startAdvertising({
  name: 'AIUI Sensor',
  serviceUUIDs: [serviceUuid],
});
```

### Server and characteristic methods

| API | Behavior |
| --- | --- |
| `server.startAdvertising(options?)` | Starts BLE advertising. |
| `server.stopAdvertising()` | Stops advertising. |
| `server.getService(uuid)` | Returns a configured local service. |
| `service.getCharacteristic(uuid)` | Returns a configured characteristic. |
| `characteristic.updateValue(value, options?)` | Updates the value and notifies subscribed devices when applicable. |
| `server.close()` | Stops and releases the server. |

Characteristic events are `readrequest`, `writerequest`, `subscribe`, and `unsubscribe`. Respond to read requests with `event.respondWith(value)`; read written bytes from `event.value` and complete writes with `event.respondWith()`.

The service structure is fixed after `openGattServer()` succeeds. Close and recreate the server to change it. Stop advertising and close it during Worker cleanup.

## `AbsoluteOrientationSensor`

### Constructor

- `new AbsoluteOrientationSensor(options?)`

### Constructor options

- `frequency?: number`

### Properties

- `quaternion`
- `timestamp`
- `activated`
- `hasReading`

### Methods

- `start()`
- `stop()`

### Event behavior

- `AbsoluteOrientationSensor` inherits from `EventTarget`.
- Supported event names are `activate`, `reading`, and `error`.
- `activate` exposes `sessionId`.
- `reading` exposes `sessionId`, `x`, `y`, `z`, `w`, `quaternion`, and `timestamp`.
- `error` exposes `sessionId`, `error`, and `message`.

### Behavior notes

- `frequency` is forwarded to the host as a best-effort hint.
- A fresh instance starts with `activated === false` and `hasReading === false`.
- `quaternion` and `timestamp` stay `null` until the first successful reading arrives.
- The first successful reading flips `activated` to `true` and `hasReading` to `true`.
- `stop()` sets `activated` back to `false` but keeps the last successful reading cached.
- `stop()` is a no-op when the instance is already idle.
- `quaternion` is exposed in `[x, y, z, w]` order.

## `Gyroscope`

### Constructor

- `new Gyroscope(options?)`

### Constructor options

- `frequency?: number`

### Properties

- `x`
- `y`
- `z`
- `timestamp`
- `activated`
- `hasReading`

### Methods

- `start()`
- `stop()`

### Event behavior

- `Gyroscope` inherits from `EventTarget`.
- Supported event names are `activate`, `reading`, and `error`.
- `activate` exposes `sessionId`.
- `reading` exposes `sessionId`, `x`, `y`, `z`, and `timestamp`.
- `error` exposes `sessionId`, `error`, and `message`.

### Behavior notes

- `frequency` is forwarded to the host as a best-effort hint.
- A fresh instance starts with `activated === false` and `hasReading === false`.
- `x`, `y`, `z`, and `timestamp` stay `null` until the first successful reading arrives.
- The first successful reading flips `activated` to `true` and `hasReading` to `true`.
- `stop()` sets `activated` back to `false` but keeps the last successful reading cached.
- `stop()` is a no-op when the instance is already idle.
