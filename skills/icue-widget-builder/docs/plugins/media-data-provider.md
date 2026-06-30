# Media Data Provider

Media player data transfer between iCUE and HTML widgets: currently playing track information and playback controls.

## Overview

-   **Module name**: `widgetbuilder.mediadataprovider`
-   **Plugin name**: `Media`
-   **Version**: `1.0`

Manifest entry:

```json
"required_plugins": [
  "widgetbuilder.mediadataprovider:Media:1.0"
]
```

## Properties

| Property   | Type     | Description         |
| ---------- | -------- | ------------------- |
| `songName` | `string` | Current song name   |
| `artist`   | `string` | Current artist name |

## Methods

### `getSongName(requestId)`

Gets the current song name asynchronously.

| Parameter   | Type  | Description                |
| ----------- | ----- | -------------------------- |
| `requestId` | `int` | Request ID for correlation |

**Response**: `string`

### `getArtist(requestId)`

Gets the current artist name asynchronously.

| Parameter   | Type  | Description                |
| ----------- | ----- | -------------------------- |
| `requestId` | `int` | Request ID for correlation |

**Response**: `string`

### `triggerPlayPause()`

Toggles play/pause state.

### `triggerNextTrack()`

Skips to the next track.

### `triggerPreviousTrack()`

Returns to the previous track.

## Signals

### `asyncResponse(requestId, value)`

Emitted when an async method completes.

| Parameter   | Type  | Description         |
| ----------- | ----- | ------------------- |
| `requestId` | `int` | Original request ID |
| `value`     | `var` | Response value      |

## SimpleMediaApiWrapper

Promise-based wrapper for the Media plugin. Converts the callback-based Qt async API into Promises.

### Important: Use Local Wrapper Files

Copy `common/plugins/` from the documentation bundle into your widget folder and include wrappers via local `<script src>` paths.

Do not reference iCUE installation paths (for example `../common/plugins/...`) in third-party widgets.

### Initialization

```javascript
const api = new SimpleMediaApiWrapper(window.plugins.Mediadataprovider);
```

| Parameter    | Type      | Default  | Description                                           |
| ------------ | --------- | -------- | ----------------------------------------------------- |
| `plugin`     | `object`  | -        | Plugin instance (`window.plugins.Mediadataprovider`)  |
| `timeoutMs`  | `number`  | `5000`   | Request timeout (ms)                                  |

### Methods

#### `getSongName()`

Returns `Promise<string>` with the current song name.

#### `getArtist()`

Returns `Promise<string>` with the current artist name.

### Required local files

```html
<script src="common/plugins/IcueWidgetApiWrapper.js"></script>
<script src="common/plugins/SimpleMediaApiWrapper.js"></script>
```

### Example

`manifest.json`

```json
"required_plugins": [
  "widgetbuilder.mediadataprovider:Media:1.0"
]
```

`index.html`

```javascript
// After including IcueWidgetApiWrapper and SimpleMediaApiWrapper in <head>:
const mediaApi = new SimpleMediaApiWrapper(window.plugins.Mediadataprovider);

async function displayCurrentTrack() {
    const [songName, artist] = await Promise.all([
        mediaApi.getSongName(),
        mediaApi.getArtist(),
    ]);
    console.log(`Now playing: ${songName} by ${artist}`);
}

// Playback controls (synchronous — call directly on the plugin)
window.plugins.Mediadataprovider.triggerPlayPause();
window.plugins.Mediadataprovider.triggerNextTrack();
window.plugins.Mediadataprovider.triggerPreviousTrack();
```
