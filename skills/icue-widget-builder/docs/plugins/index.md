# HTML Widget Plugins

HTML widget plugins extend iCUE widget functionality, and are registered using the `required_plugins` property in manifest.json.

## Supported Plugins

| Plugin                                              | Module name                       | Plugin name | Version | Description                        |
| --------------------------------------------------- | --------------------------------- | ----------- | ------- | ---------------------------------- |
| [Sensors Data Provider](./sensors-data-provider.md) | widgetbuilder.sensorsdataprovider | Sensors     | 1.0     | Sensor data from connected devices |
| [Media Data Provider](./media-data-provider.md)     | widgetbuilder.mediadataprovider   | Media       | 1.1     | Media playback control             |
| [Link Provider](./link-provider.md)                 | widgetbuilder.linkprovider        | Url         | 1.0     | Open links in the system browser   |
| [Stream Deck Provider](./stream-deck.md)            | widgetbuilder.streamdeck          | StreamDeck  | 1.2     | Stream Deck integration            |
| [FPS Data Provider](./fps-data-provider.md)         | widgetbuilder.fpsdataprovider     | Fps         | 1.0     | FPS data and current process name  |
| [Device Action Provider](./device-action-provider.md) | widgetbuilder.deviceactionprovider | DeviceAction | 1.0     | Dial and key action events from the device |
| [File System Provider](./file-system-provider.md)   | widgetbuilder.filesystemprovider  | FileSystem  | 1.0     | File and directory watching with permission enforcement |
| [Notifications Provider](./notifications-provider.md) | widgetbuilder.notificationsprovider | Notifications | 1.0 | Incoming system notification count |

## Declaration

Plugins are declared in the manifest.json file in the "required_plugins" section.

Single plugin:

```json title="manifest.json"
"required_plugins": [
    "widgetbuilder.sensorsdataprovider:Sensors:1.0"
]
```

Multiple plugins:

```json title="manifest.json"
"required_plugins": [
    "widgetbuilder.sensorsdataprovider:Sensors:1.0",
    "widgetbuilder.mediadataprovider:Media:1.1"
]
```

## Plugin Interface

### Initialization Check

Boolean variable indicating plugin load status: `plugin<<module_name>>_initialized`

Examples:

- `pluginSensorsdataprovider_initialized`
- `pluginMediadataprovider_initialized`
- `pluginLinkprovider_initialized`
- `pluginFpsdataprovider_initialized`
- `pluginStreamdeck_initialized`
- `pluginDeviceactionprovider_initialized`
- `pluginFilesystemprovider_initialized`
- `pluginNotificationsprovider_initialized`

### Initialization Callback

```javascript
plugin<<module_name>>Events = {
    "onInitialized": <<callback_function>>
};
```

Sensors plugin example:

```javascript
pluginSensorsdataproviderEvents = {
	onInitialized: onSensorsdataproviderInitialized,
};
```

Media plugin example:

```javascript
pluginMediadataproviderEvents = {
	onInitialized: onMediadataproviderInitialized,
};
```

Link plugin example:

```javascript
pluginLinkproviderEvents = {
	onInitialized: onLinkproviderInitialized,
};
```

FPS plugin example:

```javascript
pluginFpsdataproviderEvents = {
	onInitialized: onFpsdataproviderInitialized,
};
```

Stream Deck Provider example:

```javascript
pluginStreamdeckEvents = {
	onInitialized: onStreamdeckInitialized,
};
```

Device Action Provider example:

```javascript
pluginDeviceactionproviderEvents = {
	onInitialized: onDeviceactionproviderInitialized,
};
```

File System Provider example:

```javascript
pluginFilesystemproviderEvents = {
	onInitialized: onFilesystemproviderInitialized,
};
```

Notifications Provider example:

```javascript
pluginNotificationsproviderEvents = {
	onInitialized: onNotificationsproviderInitialized,
};
```

### Plugin Access

Access plugins via `window.plugins.<<module_name>>`:

- `window.plugins.Sensorsdataprovider`
- `window.plugins.Mediadataprovider`
- `window.plugins.Linkprovider`
- `window.plugins.Fpsdataprovider`
- `window.plugins.Streamdeck`
- `window.plugins.Deviceactionprovider`
- `window.plugins.Filesystemprovider`
- `window.plugins.Notificationsprovider`

Each plugin has its own interface documented in the plugin-specific documentation.

## Plugin Permissions

> **Skill note:** a summary assembled from the per-plugin pages and the [Permissions](../permissions.md) reference.

Some plugins reach resources outside the widget sandbox and are gated by entries in the manifest `permissions` array
(declared next to `required_plugins`):

| Plugin                                   | Permission to declare                                                     |
| ---------------------------------------- | ------------------------------------------------------------------------- |
| [Stream Deck Provider](./stream-deck.md) | `{ "type": "application", "application": "Stream Deck" }`                 |
| [File System Provider](./file-system-provider.md) | `file` (single watched file) and/or `folder` (watched directory) |

The Sensors, Media, Link, FPS, Device Action, and Notifications provider pages do not document any permission
requirement. Do not invent permission entries for them.
