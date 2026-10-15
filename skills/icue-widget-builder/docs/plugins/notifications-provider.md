# Notification Provider

Notification data transfer between iCUE and HTML widgets: incoming notification count

## Overview

-   **Module name**: `widgetbuilder.notificationsprovider`
-   **Plugin name**: `Notifications`
-   **Version**: `1.0`

Manifest entry:

```json
"required_plugins": [
  "widgetbuilder.notificationsprovider:Notifications:1.0"
]
```

Runtime objects:

- Initialization flag: `pluginNotificationsprovider_initialized`
- Initialization callback map: `pluginNotificationsproviderEvents = { "onInitialized": onNotificationsproviderInitialized }`
- Plugin access: `window.plugins.Notificationsprovider`

## Methods

### `getNotificationCount()`

Get number of incoming system notifications.

**Response**: `int` - number of incoming notifications

## Signals

### `notificationCountChanged()`

Emitted when number of incoming system notifications changes.

## SimpleNotificationsApiWrapper

Promise-based wrapper for the Notifications plugin. Converts the callback-based Qt async API into
Promises.

### Important: Use Local Wrapper Files

Copy `common/plugins/` from the documentation bundle into your widget folder and include
wrappers via local `<script src>` paths.

Do not reference iCUE installation paths (for example `../common/plugins/...`) in
third-party widgets.

### Initialization

```javascript
const api = new SimpleNotificationsApiWrapper(window.plugins.Notificationsprovider);
```

| Parameter   | Type     | Default | Description                                              |
| ----------- | -------- | ------- | -------------------------------------------------------- |
| `plugin`    | `object` | -       | Plugin instance (`window.plugins.Notificationsprovider`) |
| `timeoutMs` | `number` | `5000`  | Request timeout (ms)                                     |

### Methods

All methods return a `Promise`.

| Method                   | Returns            | Description                             |
| ------------------------ | ------------------ | --------------------------------------- |
| `getNotificationCount()` | `Promise<int>`     | Number of incoming system notifications |

### Required local files

```html
<script src="common/plugins/IcueWidgetApiWrapper.js"></script>
<script src="common/plugins/SimpleNotificationsApiWrapper.js"></script>
```

### Example

`manifest.json`

```json
"required_plugins": [
  "widgetbuilder.notificationsprovider:Notifications:1.0"
]
```

`index.html`

```javascript
// After including IcueWidgetApiWrapper and SimpleNotificationsApiWrapper in <head>:
const notificationsApi = new SimpleNotificationsApiWrapper(window.plugins.Notificationsprovider);

async function displayNotifications() {
	const notificationCount = await notificationsApi.getNotificationCount();
	console.warn("Notifications count:", notificationCount);
}
displayNotifications();
```
