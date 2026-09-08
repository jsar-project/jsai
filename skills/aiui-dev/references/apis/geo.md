# AIUI Geography and Location API Reference

## Permission

Declare location access in `app.json`:

```json
{ "permissions": ["GEOLOCATION"] }
```

## `navigator.geolocation`

The runtime supplies this object; agents do not construct it.

### `getCurrentPosition(success, error?, options?)`

Requests one position and reports it through callbacks.

### `watchPosition(success, error?, options?)`

Returns a numeric watch ID and reports subsequent position changes. Release it with `clearWatch(watchId)`.

### `clearWatch(watchId)`

Stops the corresponding position watch.

### Position options

| Field | Type | Meaning |
| --- | --- | --- |
| `enableHighAccuracy` | `boolean` | Prefer more accurate positioning; defaults to `false` and may use more power. |
| `timeout` | `number` | Maximum wait in milliseconds. |
| `maximumAge` | `number` | Maximum acceptable cached-position age in milliseconds. |

### Position result

`position.coords` contains `latitude`, `longitude`, `accuracy`, and nullable `altitude`, `altitudeAccuracy`, `heading`, and `speed`. `position.timestamp` is the acquisition time.

Errors use codes `1` (`PERMISSION_DENIED`), `2` (`POSITION_UNAVAILABLE`), and `3` (`TIMEOUT`).

```javascript
const watchId = navigator.geolocation.watchPosition(
  (position) => {
    const { latitude, longitude } = position.coords;
    console.log(latitude, longitude);
  },
  (error) => console.error(error.code, error.message),
  { enableHighAccuracy: true, timeout: 10_000 },
);

// During cleanup:
navigator.geolocation.clearWatch(watchId);
```

## `GPXDocument`

Available globally and as a named export from `'gpx'`:

```javascript
import { GPXDocument } from 'gpx';
```

### Construction and parsing

- `new GPXDocument(input?)`
- `GPXDocument.from(input)`
- `GPXDocument.parse(input)`

Accepted input is GPX XML text, `Blob`, `ArrayBuffer`, a typed array/`BufferSource`, or another `GPXDocument`. `from()` and `parse()` have the same behavior.

### Route methods

| Method | Result |
| --- | --- |
| `setStartPoint(point)` | Sets the start point. |
| `setEndPoint(point)` | Sets the end point. |
| `addWaypoint(point)` | Adds a waypoint. |
| `appendTrackPoint(point)` | Appends a track point. |
| `clearTrack()` | Removes track points but keeps start, end, and waypoints. |
| `toString()` | Returns GPX XML text. |
| `toBlob()` | Returns a Blob with `application/gpx+xml`. |

A point requires numeric `latitude` and `longitude`; optional fields are `elevation`, `time`, and `name`. Read-only `bounds` is `null` when there are no points, otherwise it contains minimum and maximum latitude/longitude.

```javascript
const route = new GPXDocument();
route.setStartPoint({ latitude: 30.2741, longitude: 120.1551 });
route.appendTrackPoint({
  latitude: 30.2792,
  longitude: 120.1618,
  time: new Date().toISOString(),
});
const xml = route.toString();
```
