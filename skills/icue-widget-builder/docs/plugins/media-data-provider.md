# Media Data Provider

Media player data transfer between iCUE and HTML widgets: currently playing track information and playback controls.

## Overview

-   **Module name**: `widgetbuilder.mediadataprovider`
-   **Plugin name**: `Media`
-   **Version**: `1.1`

Manifest entry:

```json
"required_plugins": [
  "widgetbuilder.mediadataprovider:Media:1.1"
]
```

## Properties

| Property    | Type     | Description                                                              |
| ----------- | -------- | ----------------------------------------------------------------------- |
| `songName`  | `string` | Current song name                                                       |
| `artist`    | `string` | Current artist name                                                     |
| `album`     | `string` | Current album name                                                      |
| `thumbnail` | `string` | Album art as a base64 PNG data URL (`data:image/png;base64,...`), or an empty string when unavailable |
| `isPlaying` | `bool`   | `true` while media is playing, `false` when paused or stopped           |

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

### `getAlbum(requestId)`

Gets the current album name asynchronously.

| Parameter   | Type  | Description                |
| ----------- | ----- | -------------------------- |
| `requestId` | `int` | Request ID for correlation |

**Response**: `string`

### `getThumbnail(requestId)`

Gets the current album art asynchronously as a base64 PNG data URL. The value can be assigned directly to an `<img>` `src` or a CSS `background-image`. Returns an empty string when no album art is available.

| Parameter   | Type  | Description                |
| ----------- | ----- | -------------------------- |
| `requestId` | `int` | Request ID for correlation |

**Response**: `string`

### `getIsPlaying(requestId)`

Gets the current playback state asynchronously. Returns `true` while media is playing, `false` when paused or stopped.

| Parameter   | Type  | Description                |
| ----------- | ----- | -------------------------- |
| `requestId` | `int` | Request ID for correlation |

**Response**: `bool`

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

### Property change signals

Emitted when the corresponding property changes. Connect to them to refresh the widget when the current track updates.

| Signal              | Related property |
| ------------------- | ---------------- |
| `songNameChanged`   | `songName`       |
| `artistChanged`     | `artist`         |
| `albumChanged`      | `album`          |
| `thumbnailChanged`  | `thumbnail`      |
| `isPlayingChanged`  | `isPlaying`      |

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

| Method            | Returns             | Description                                                       |
| ----------------- | ------------------- | --------------------------------------------------------------- |
| `getSongName()`   | `Promise<string>`   | Currently playing track title                                   |
| `getArtist()`     | `Promise<string>`   | Currently playing artist name                                   |
| `getAlbum()`      | `Promise<string>`   | Currently playing album name                                    |
| `getThumbnail()`  | `Promise<string>`   | Album art as a base64 PNG data URL, or an empty string when unavailable |
| `getIsPlaying()`  | `Promise<bool>`     | `true` while media is playing, `false` when paused or stopped   |

### Required local files

```html
<script src="common/plugins/IcueWidgetApiWrapper.js"></script>
<script src="common/plugins/SimpleMediaApiWrapper.js"></script>
```

### Example

`manifest.json`

```json
"required_plugins": [
  "widgetbuilder.mediadataprovider:Media:1.1"
]
```

`index.html`

```javascript
// After including IcueWidgetApiWrapper and SimpleMediaApiWrapper in <head>:
const mediaApi = new SimpleMediaApiWrapper(window.plugins.Mediadataprovider);

async function displayCurrentTrack() {
    const [songName, artist, album, thumbnail] = await Promise.all([
        mediaApi.getSongName(),
        mediaApi.getArtist(),
        mediaApi.getAlbum(),
        mediaApi.getThumbnail(),
    ]);
    console.log(`Now playing: ${songName} by ${artist} (${album})`);

    // thumbnail is a base64 PNG data URL, usable directly as an image source
    if (thumbnail) {
        document.getElementById("albumArt").src = thumbnail;
    }
}

// React to track changes via property change signals
window.plugins.Mediadataprovider.songNameChanged.connect(displayCurrentTrack);
window.plugins.Mediadataprovider.artistChanged.connect(displayCurrentTrack);
window.plugins.Mediadataprovider.albumChanged.connect(displayCurrentTrack);
window.plugins.Mediadataprovider.thumbnailChanged.connect(displayCurrentTrack);

// Keep the play/pause button in sync with the real playback state
function updatePlaybackState() {
    const playing = window.plugins.Mediadataprovider.isPlaying;
    document.getElementById("playPauseButton").textContent = playing ? "Pause" : "Play";
}
window.plugins.Mediadataprovider.isPlayingChanged.connect(updatePlaybackState);

// Playback controls (synchronous — call directly on the plugin)
window.plugins.Mediadataprovider.triggerPlayPause();
window.plugins.Mediadataprovider.triggerNextTrack();
window.plugins.Mediadataprovider.triggerPreviousTrack();
```
