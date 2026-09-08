# AIUI Media API Reference

This file documents the verified media playback APIs available to AIUI agent code.

- Common scope, entry points, and authoring rules live in [apis.md](./index.md).
- Keep examples aligned with the current local-file-focused implementation.

## `AudioPlayer`

### Constructor

- `new AudioPlayer(options?)`

### Common properties

- `src`
- `autoplay`
- `loop`
- `volume`
- `currentTime`
- `paused`

### Common methods

- `play()`
- `pause()`
- `stop()`
- `seek(position)`
- `destroy()`
- `append(buffer)`
- `finish()`

### Behavior notes

- `AudioPlayer` is the general-purpose playback API for full audio files and streaming audio data.
- For file playback, set `player.src` to either a relative local path or a project-root absolute path such as `/assets/intro.ogg`.
- Local path handling explicitly covers leading-slash asset paths in the agent package.
- Network URLs are supported for `src`, but local packaged assets remain the preferred choice for bundled media.
- Streaming playback is enabled through the constructor `options` object and currently documented for `format: 'pcm'` and `format: 'ogg_opus'`.
- Use `append()` to push streaming chunks and `finish()` when the stream ends.

### Example

```javascript
import { AudioPlayer } from 'audio';

const player = new AudioPlayer();
player.src = '/assets/intro.ogg';
player.loop = true;
player.play();
```

## `Sound`

### Constructor

- `new Sound(src)`

### Properties

- `volume`

### Methods

- `play()`
- `stop()`
- `destroy()`

### Behavior notes

- `src` must be a non-empty local file path.
- Remote URLs such as `http://` and `https://` are rejected.
- The source is bound during construction so the instance is ready for replay-oriented playback.
- Local sound effect paths can be relative or project-root absolute, for example `/assets/click.wav`.
- `volume` is a read/write number.
- `play()` stops any current playback on the instance and starts again from the beginning.
- `Sound` supports local files only.
- `Sound` does not expose `src` mutation, seeking, streaming, or event callbacks.

### Error behavior

- After `destroy()`, later method calls throw.

## Web Audio

Web Audio classes are global and are also named exports from `'audio'`. Start with `new AudioContext(options?)`; its initial state may be `suspended`, so call `resume()` before expecting processing.

### Context creation methods

| Method | Returns | Purpose |
| --- | --- | --- |
| `createBuffer(channels, length, sampleRate)` | `AudioBuffer` | Creates PCM storage. |
| `createBufferSource()` | `AudioBufferSourceNode` | Plays an `AudioBuffer`. |
| `createOscillator()` | `OscillatorNode` | Generates sine, square, sawtooth, or triangle audio. |
| `createGain()` | `GainNode` | Adjusts volume. |
| `createBiquadFilter()` | `BiquadFilterNode` | Filters audio. |
| `createAnalyser()` | `AnalyserNode` | Reads waveform and frequency data. |
| `createMediaStreamSource(mediaStream)` | `MediaStreamAudioSourceNode` | Reads an audio track from a media stream. |
| `decodeAudioData(data, success?, error?)` | `Promise<AudioBuffer>` | Decodes encoded audio bytes. |

`AudioContext` also exposes `state`, `currentTime`, `sampleRate`, `destination`, `resume()`, `suspend()`, and `close()`.

### Audio graph

All nodes inherit `connect(destination, output?, input?)` and `disconnect()`. A common graph is source → processing nodes → `context.destination`. Analyser-only microphone input should not connect to the destination because speaker playback can cause feedback.

### `MediaStreamAudioSourceNode`

Create it with `context.createMediaStreamSource(stream)` or `new MediaStreamAudioSourceNode(context, { mediaStream: stream })`. `mediaStream` is read-only and `numberOfInputs` is `0`.

- A stream without an audio track throws `InvalidStateError`.
- A value that is not a `MediaStream` throws `TypeError`.
- `track.enabled = false` temporarily silences input; setting it back to `true` resumes it.
- `track.stop()` ends the track and the node then outputs silence.

```javascript
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
const context = new AudioContext();
const source = context.createMediaStreamSource(stream);
const analyser = context.createAnalyser();
const waveform = new Float32Array(analyser.fftSize);

source.connect(analyser);
await context.resume();
analyser.getFloatTimeDomainData(waveform);

// During cleanup:
source.disconnect();
stream.getTracks().forEach((track) => track.stop());
await context.close();
```

Declare `RECORD_AUDIO` in `app.json` and call `getUserMedia()` from a valid user interaction.

### Key node details

- `AudioBuffer`: `numberOfChannels`, `length`, `sampleRate`, `duration`, `getChannelData()`, `copyFromChannel()`, `copyToChannel()`.
- `AudioBufferSourceNode`: `buffer`, `playbackRate`, `detune`, `loop`, `loopStart`, `loopEnd`, `start(when?, offset?, duration?)`, `stop(when?)`.
- `GainNode`: `gain: AudioParam`.
- `OscillatorNode`: `type`, `frequency`, `detune`, `start()`, `stop()`.
- `BiquadFilterNode`: `type`, `frequency`, `detune`, `Q`, `gain`, `getFrequencyResponse()`.
- `AnalyserNode`: `fftSize`, `frequencyBinCount`, decibel range, smoothing, and float/byte waveform/frequency getters.
- `AudioParam`: `value`, automation scheduling methods, and cancellation methods.

## Media Capture and Recording

### `navigator.mediaDevices`

- `getUserMedia(constraints): Promise<MediaStream>`
- `enumerateDevices(): Promise<MediaDeviceInfo[]>`
- `getSupportedConstraints(): MediaTrackSupportedConstraints`

Audio/video constraints may be booleans or objects. Supported request fields include `deviceId`, `sampleRate`, `channelCount`, `echoCancellation`, `facingMode`, `width`, `height`, and `frameRate`. Read the final device-selected values from `track.getSettings()`.

### `MediaStream`

Use `getTracks()`, `getAudioTracks()`, `getVideoTracks()`, and `getTrackById(id)`. Stop tracks when finished.

### `MediaRecorder`

`new MediaRecorder(stream, options?)` records a stream. Check formats with `MediaRecorder.isTypeSupported(mimeType)`. Key members include `state`, `mimeType`, `start(timeslice?)`, `stop()`, `pause()`, `resume()`, `requestData()`, `ondataavailable`, `onstart`, `onstop`, `onpause`, `onresume`, and `onerror`.

Supported media types currently include `audio/wav`, `audio/ogg;codecs=opus`, `video/webm;codecs=vp8,opus`, and `video/mp4`.

### `ImageCapture`

Construct with a video `MediaStreamTrack`. `takePhoto(options?)` returns an encoded `Blob`; `grabFrame()` returns an in-memory `ImageBitmap`.
