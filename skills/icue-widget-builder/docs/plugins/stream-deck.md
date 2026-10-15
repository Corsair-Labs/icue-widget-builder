# Stream Deck Provider

Provider for integrating HTML widgets with Stream Deck: connects to Stream Deck devices, manages virtual device layout, and sends key events.

Note: Stream Deck Provider is available only on Windows.

## Revision History

- **Rev 1.1**
  - Added documentation for Stream Deck signals: `virtualDeviceCreated`,
    `buttonIconUpdated`, `streamdeckUnreachable`, `authenticationRequired`,
    `authenticationRejected`.
  - Documented `reconnectStreamDeck(widgetId)` behavior, including when a new
    virtual device is created vs. reusing an existing one.
  - Added `disconnectStreamDeck(widgetId)` method description and QML example
    wiring for all error-related signals.
- **Rev 1.2**
  - Updated `connectStreamDeck` behavior: it no longer creates a new virtual device when an empty `deviceId` is provided.
    To create a virtual device, use the `stream-deck-virtual-device` control manually.
  - When a widget is deleted, its virtual device is no longer deleted.
    To delete a virtual device, navigate to the Stream Deck application and delete it there.
  - Added documentation for Stream Deck signal `streamdeckAuthenticated`.

## Overview

-   **Module name**: `widgetbuilder.streamdeck`
-   **Plugin name**: `StreamDeck`
-   **Version**: `1.2`

Manifest entry:

```json
"required_plugins": [
  "widgetbuilder.streamdeck:StreamDeck:1.2"
],
"permissions": [
  { "type": "application", "application": "Stream Deck" }
]
```

Runtime objects:

- Initialization flag: `pluginStreamdeck_initialized`
- Initialization callback map: `pluginStreamdeckEvents = { "onInitialized": onStreamdeckInitialized }`
- Plugin access: `window.plugins.Streamdeck`

All methods are synchronous from JavaScript perspective but operate on an asynchronous RPC connection to the Stream Deck provider. Device creation, icon updates, and error conditions are delivered via signals.

## Required Permission

> **Skill note:** this section is not in the upstream plugin page. It is derived from the
> [Permissions](../permissions.md#application) reference, which lists Stream Deck as a supported `application`
> permission and `widgetbuilder.streamdeck:StreamDeck:1.2` as its required plugin.

The plugin talks to the external Stream Deck desktop application, so the widget manifest must declare the
`application` permission for it:

```json
"permissions": [
  { "type": "application", "application": "Stream Deck" }
]
```

- The application name is the permission scope and is matched case-insensitively (`"Stream Deck"` = `"stream deck"`).
- Pre-declared permissions are shown to the user at install time. If the entry is missing, iCUE asks the user at
  runtime, and the plugin must not connect to Stream Deck until the user approves.
- The user can revoke the permission at any time. Treat a revoked or denied permission like a lost connection:
  the plugin disconnects, and retries once the user approves again. Do not show a permanent "broken" state;
  show a recoverable "Stream Deck access is not allowed" hint instead.
- Declare one `application` permission per external application the widget uses; do not reuse it for other apps.

## Methods

### `connectStreamDeck(widgetId, deviceId, columns, rows)`

Connects the widget to an existing virtual Stream Deck device with the specified grid size.

| Parameter   | Type     | Description                                  |
| ----------- | -------- | -------------------------------------------- |
| `widgetId`  | `string` | Unique widget identifier provided by iCUE    |
| `deviceId`  | `string` | Target Stream Deck device identifier         |
| `columns`   | `int`    | Number of columns in the virtual key matrix  |
| `rows`      | `int`    | Number of rows in the virtual key matrix     |

On success, `virtualDeviceCreated(widgetId, deviceId)` is emitted once the virtual device is ready.

On connection or authentication errors, one or more of the following signals may
be emitted (see the **Signals** section below):

- `streamdeckAuthenticated(widgetId)` – when authentication succeeds.
- `streamdeckUnreachable(widgetId)` – when the Stream Deck provider cannot be
  reached or the connection drops.
- `authenticationRequired(widgetId)` – when the user must authenticate before
  the widget can interact with Stream Deck.
- `authenticationRejected(widgetId)` – when authentication fails or is rejected.

---

### `reconnectStreamDeck(widgetId)`

Requests reconnection for a previously connected widget. The method is asynchronous and
restarts the underlying connection logic for the given `widgetId`, attempting to reuse
the existing virtual device when possible. If the original virtual device no longer
exists on the Stream Deck side, a new one is created during reconnection (see signal
behavior below).

| Parameter  | Type     | Description                               |
| ---------- | -------- | ----------------------------------------- |
| `widgetId` | `string` | Widget identifier used during connect     |

Use this when the Stream Deck process was restarted or the connection was lost and
re-established. Calling `reconnectStreamDeck` multiple times for the same `widgetId`
is safe; as long as the original virtual device still exists, no extra devices are
created.

Signal behavior:

- If the existing virtual device for `widgetId` is still present on the Stream Deck
  side, **`virtualDeviceCreated(widgetId, deviceId)` is not emitted again**. The
  original `deviceId` remains valid. When the device is opened after reconnection,
  the plugin will re-emit `buttonIconUpdated(widgetId, buttonIndex, iconDataUrl)`
  for all known buttons.

- If the stored virtual device no longer exists (for example, the Stream Deck app
  lost its state), a **new virtual device is created** and
  `virtualDeviceCreated(widgetId, deviceId)` is emitted again with the new
  `deviceId`.

- In case of connection problems or authentication issues, no special
  "reconnect failed" signal is emitted; instead, the regular error signals are
  used:
  - `streamdeckAuthenticated(widgetId)` when the connection transitions to the
    Authenticated state;
  - `streamdeckUnreachable(widgetId)` when the connection transitions to the
    Disconnected state;
  - `authenticationRequired(widgetId)` when user authentication is needed;

  - `authenticationRejected(widgetId)` if authentication fails.

---

### `disconnectStreamDeck(widgetId)`

Disconnects the widget from its Stream Deck session. If this is the last subscriber
for the given `widgetId`, the underlying client session and WebSocket connection are
closed.

| Parameter  | Type     | Description                           |
| ---------- | -------- | ------------------------------------- |
| `widgetId` | `string` | Widget identifier used during connect |

Typical usage:

- Call `disconnectStreamDeck(widgetId)` when the widget is removed or its QML/HTML
  component is destroyed.
- When multiple views share the same `widgetId` (for example, dashboard + mimic), the
  Stream Deck session remains active until the last view disconnects.

---

### `updateVirtualDeviceSize(widgetId, columns, rows)`

Updates the size of the virtual device grid without reconnecting.

| Parameter  | Type     | Description                                  |
| ---------- | -------- | -------------------------------------------- |
| `widgetId` | `string` | Widget identifier                            |
| `columns`  | `int`    | New number of columns                        |
| `rows`     | `int`    | New number of rows                           |

The provider keeps existing connection and only resizes the virtual device.

---

### `sendKeyPress(widgetId, buttonIndex, pressed)`

Sends a key press or release event for a virtual button.

| Parameter     | Type     | Description                                       |
| ------------- | -------- | ------------------------------------------------- |
| `widgetId`    | `string` | Widget identifier                                |
| `buttonIndex` | `int`    | Zero-based button index in the virtual key grid  |
| `pressed`     | `bool`   | `true` for press, `false` for release            |

The grid is traversed in **row-major** order:

- Rows are numbered from top to bottom starting at `0`.
- Columns are numbered from left to right starting at `0`.
- The index is computed as:

  `buttonIndex = row * columns + column`

For example, for a `5 × 3` grid (`columns = 5`, `rows = 3`):

- `buttonIndex = 0` → row `0`, column `0` (top-left)
- `buttonIndex = 4` → row `0`, column `4` (top-right)
- `buttonIndex = 5` → row `1`, column `0` (first button of the second row)
- `buttonIndex = 14` → row `2`, column `4` (bottom-right)

Use this from an interactive widget when the user taps or releases a virtual key.

## Properties

### `connectionStatus`

Current state of the connection to the Stream Deck application, reported as a `string`.
The value belongs to the widget passed to `connectStreamDeck(widgetId, ...)`, statuses of
other widgets are ignored. The value is updated automatically and the
`connectionStatusChanged()` notify signal is emitted on every change. The signal carries
no arguments, read the property to get the current value.

| Value                      | Description                                          |
| -------------------------- | ---------------------------------------------------- |
| `"Disconnected"`           | No connection to the Stream Deck application         |
| `"Connecting"`             | Connection is being established                      |
| `"AuthRequested"`          | Waiting for the user to approve the iCUE connection  |
| `"Authenticated"`          | Connection is approved                               |
| `"DeviceOpened"`           | Virtual device is opened and ready                   |
| `"AuthenticationRejected"` | The iCUE connection was not approved                 |

```javascript
const plugin = window.plugins.Streamdeck;
plugin.connectionStatusChanged.connect(function () {
	// Update the widget UI according to plugin.connectionStatus
});
```

## Signals

### `virtualDeviceCreated(widgetId, deviceId)`

Emitted when a virtual Stream Deck device has been created and is ready.

| Parameter  | Type     | Description                      |
| ---------- | -------- | -------------------------------- |
| `widgetId` | `string` | Widget identifier                |
| `deviceId` | `string` | Connected Stream Deck device id  |

---

### `buttonIconUpdated(widgetId, buttonIndex, iconDataUrl)`

Emitted when an icon for a specific virtual button changes.

| Parameter     | Type     | Description                             |
| ------------- | -------- | --------------------------------------- |
| `widgetId`    | `string` | Widget identifier                       |
| `buttonIndex` | `int`    | Button index whose icon was updated    |
| `iconDataUrl` | `string` | Icon image as a `data:` URL (PNG, etc) |

---

### `streamdeckAuthenticated(widgetId)`

Emitted when a connection with the Stream Deck application is established.

| Parameter  | Type     | Description       |
| ---------- | -------- | ----------------- |
| `widgetId` | `string` | Widget identifier |

---

### `streamdeckUnreachable(widgetId)`

Emitted when the Stream Deck provider cannot be reached or the connection is lost.

| Parameter  | Type     | Description       |
| ---------- | -------- | ----------------- |
| `widgetId` | `string` | Widget identifier |

---

### `authenticationRequired(widgetId)`

Emitted when authentication is required before the widget can interact with Stream Deck.

| Parameter  | Type     | Description       |
| ---------- | -------- | ----------------- |
| `widgetId` | `string` | Widget identifier |

---

### `authenticationRejected(widgetId)`

Emitted when authentication fails or is rejected.

| Parameter  | Type     | Description       |
| ---------- | -------- | ----------------- |
| `widgetId` | `string` | Widget identifier |

## Example

`manifest.json`

```json
"required_plugins": [
  "widgetbuilder.streamdeck:StreamDeck:1.2"
],
"permissions": [
  { "type": "application", "application": "Stream Deck" }
]
```

`index.html`

Add the [stream-deck-virtual-device](../controls/stream-deck-virtual-device.md) control to the HTML `<head>` so the user can select a virtual device:

```html
<meta
    name="x-icue-property"
    content="virtualDevice"
    data-label="tr('Virtual Device')"
    data-type="stream-deck-virtual-device"
/>
```

Then use the selected `virtualDevice` value in your script:

```javascript
// Initialize plugin events for async signals (recommended)
pluginStreamdeckEvents = {
	"onInitialized": function onStreamdeckInitialized() {
		if (!window.plugins || !window.plugins.Streamdeck) {
			return;
		}

		const plugin = window.plugins.Streamdeck;

		// Handle successful virtual device creation
		plugin.virtualDeviceCreated.connect(function (widgetId, deviceId) {
			// Example: hide an error overlay and cache deviceId if needed
		});

		// Handle connection loss
		plugin.streamdeckUnreachable.connect(function (widgetId) {
			// Example: show an error overlay in the widget
		});

		// Plugin is ready; you can safely call methods here
		connectStreamDeckPlugin();
	}
};

function connectStreamDeckPlugin() {
	if (window.plugins && window.plugins.Streamdeck && pluginStreamdeck_initialized) {
		const widgetId = iCUE.widgetId; // Provided by iCUE global object
		const deviceId = virtualDevice; // Provided by the stream-deck-virtual-device control
		const columns = 5;
		const rows = 3;

		window.plugins.Streamdeck.connectStreamDeck(widgetId, deviceId, columns, rows);
	}
}

// Called by your UI when a virtual button is pressed or released
function onVirtualButtonPressed(buttonIndex, pressed) {
	if (window.plugins && window.plugins.Streamdeck && pluginStreamdeck_initialized) {
		window.plugins.Streamdeck.sendKeyPress(iCUE.widgetId, buttonIndex, pressed);
	}
}

// Example: bind UI events to virtual button presses
const button = document.getElementById("streamdeck-button-0");
if (button) {
	button.addEventListener("pointerdown", () => onVirtualButtonPressed(0, true));
	button.addEventListener("pointerup", () => onVirtualButtonPressed(0, false));
}
```

> **Note:** The iCUE widget runtime provides `iCUE.widgetId`. The selected device ID
> comes from the `virtualDevice` property defined by the
> [stream-deck-virtual-device](../controls/stream-deck-virtual-device.md) control,
> not from the `iCUE` global object.

## Settings-Panel Status Row (`x-icue-info` / `app-status`)

> **Version caveat:** the field observations below were gathered against plugin `1.0`, where `connectStreamDeck` created the
> virtual device itself. Since `1.2` the virtual device is selected with the
> [stream-deck-virtual-device](../controls/stream-deck-virtual-device.md) control and `connectStreamDeck` no longer
> creates one. Points about device creation, name collisions, and empty `deviceId` may no longer apply — verify against
> the current docs above before relying on them, and re-check the connection and retry points too.

Undocumented meta type that makes iCUE render a native "is the Stream Deck app running/connected" row in the settings panel, sourced from iCUE rather than widget JS. Any widget whose corner keys or other interactive elements depend on this plugin should declare it — without it the user cannot tell from the panel whether the plugin connected, which surfaces as "nothing happens" reports.

Declaration, attribute table, `x-icue-groups` example, and provenance: `references/widget-meta-parameters.md` → "Companion App Status".

**Default to a visible on-widget status too.** The first-party widget unconditionally shows a "Stream Deck app is not running or cannot be found" message at boot — before any signal has fired — and only clears/replaces it once `virtualDeviceCreated`, `buttonIconUpdated`, `authenticationRequired`, `authenticationRejected`, or `streamdeckUnreachable` actually arrives. A widget that only reacts to signals (and shows nothing by default) will stay silently blank if the plugin never emits any signal at all — for example when there's no Stream Deck device yet for the virtual keys to attach to.

**A brand-new virtual device has no actions, so no `buttonIconUpdated` ever arrives.** Keys stay empty placeholders until the user assigns actions to *that widget's* virtual device in the Stream Deck app. This is not a failure state — surface it as a "assign actions in Stream Deck" hint, never as a connection error, or users will read empty keys as "the widget is broken."

**`virtualDeviceCreated` may carry an empty `deviceId`.** Verified in iCUE's log (`cue.api.streamdeck_client`): the first `VirtualDeviceOpened` for a widget can come back with `deviceId = ""`, and iCUE still resolves the correct device on the next connect because it maps widget→device internally. Treat an empty `deviceId` as a successful open, and don't gate your "connected" state on it being non-empty.

**Compare widget ids brace/case-insensitively.** iCUE logs the widget id as a braced QUuid (`{f5f06033-...}`) while `uniqueId` may be injected unbraced. If a widget filters signals with raw `===` against `uniqueId`, a formatting mismatch silently drops every signal — icons never render and status never clears. Normalize (strip `{}`, lowercase) before comparing.

**Do NOT auto-reconnect on `streamdeckUnreachable`.** This is the single easiest way to break a Stream Deck widget, and it fails in a way that looks like a connection problem. `reconnectStreamDeck()` does not re-emit `virtualDeviceCreated` when the virtual device still exists, and it only re-emits `buttonIconUpdated` "for all known buttons" — but a newly created device has no actions, so there are no known buttons either. An eager auto-reconnect therefore swallows the *only* signal that would have cleared the error state, leaving the widget stuck showing "Stream Deck app is not running" while the device is connected and visible in Stream Deck. The first-party widget only ever reconnects from its explicit Connect button — do the same, and let the signals arrive on their own.

**Call `connectStreamDeck()` on every `onInitialized`, not just the first.** The first-party widget does this unconditionally. Guarding it with an "already connected" flag means that if the first connect lands before the provider is ready, the later good `onInitialized` is ignored and no session is ever established.

**`streamdeckUnreachable` can fire transiently even when Stream Deck is genuinely running.** One confirmed trigger: when stale leftover virtual devices cause `CreateVirtualDevice` name collisions (iCUE logs `Received Error from server, code = 1` → `Virtual device name already taken ... retrying with index N`), the provider emits `streamdeckUnreachable` *in the middle of an otherwise successful* connect that then completes normally ~135ms later. Track whether the device ever opened successfully and suppress the hard "app not running" message if it did. Cross-checking against iCUE's own StreamDeck-plugin logs (`%LOCALAPPDATA%/Corsair/icue_streamdeck_plugin/logs/sdp-*.log`, look for `Start connecting to StreamDeckApp` → `Successfully connected to StreamDeckApp`) showed the local handshake between iCUE and the Stream Deck app regularly taking **12-15 seconds** on a real machine (5 sampled sessions: 0s, 13s, 15s, 12s, 13s — instant only when Stream Deck was already fully warmed up). A retry/backoff window shorter than that (e.g. ~10s) will misreport a slow-but-healthy connection as unreachable. Give `streamdeckUnreachable` at least ~25-30 seconds of auto-retries (e.g. 6 retries at 4s intervals) before surfacing it to the user as a real failure.

## StreamDeck QML Wrapper

The StreamDeck QML wrapper provides a thin, declarative layer over the HTML plugin API so that
QML components can control Stream Deck devices without writing JavaScript glue code.

The wrapper is implemented in the `widgetbuilder.streamdeck` module as `StreamDeck.qml` and
internally forwards all calls to the same backend that powers the HTML plugin.

### Initialization

To use the QML wrapper, import the module and keep a single `StreamDeck` object in the QML
scope that needs to interact with Stream Deck. The wrapper exposes the same methods and
signals as the HTML plugin:

- `connectStreamDeck(widgetId, deviceId, columns, rows)`
- `reconnectStreamDeck(widgetId)`
- `disconnectStreamDeck(widgetId)`
- `updateVirtualDeviceSize(widgetId, columns, rows)`
- `sendKeyPress(widgetId, buttonIndex, pressed)`

Properties:

- `connectionStatus`

Signals:

- `virtualDeviceCreated(widgetId, deviceId)`
- `buttonIconUpdated(widgetId, buttonIndex, iconDataUrl)`
- `streamdeckAuthenticated(widgetId)`
- `streamdeckUnreachable(widgetId)`
- `authenticationRequired(widgetId)`
- `authenticationRejected(widgetId)`

### QML Example

```qml
	Item {
		id: root

		// These values are typically provided by the host QML environment,
		// for example via context properties or a view model.
		property string widgetId: "{widget-id-from-environment}"
		property string deviceId: "{streamdeck-device-id-from-environment}"

		StreamDeck {
			id: streamDeck

			onVirtualDeviceCreated: function(widgetId, deviceId) {
				// Handle initial virtual device creation
			}

			onButtonIconUpdated: function(widgetId, buttonIndex, iconDataUrl) {
				// Update button visuals in QML if needed
			}

			onStreamdeckAuthenticated: function(widgetId) {
				// Handle Stream Deck authentication
			}

			onStreamdeckUnreachable: function(widgetId) {
				// Handle connection loss, e.g. show an error overlay in QML
			}

			onAuthenticationRequired: function(widgetId) {
				// Prompt the user to authenticate before using Stream Deck
			}

			onAuthenticationRejected: function(widgetId) {
				// Inform the user that authentication failed
			}
		}

		Component.onCompleted: {
			streamDeck.connectStreamDeck(widgetId, deviceId, 5, 3);
		}

	}
```
