# Device Action Data Provider

Plugin for receiving dial and key action events from the device currently displaying the widget.

## Overview

-   **Module name**: `widgetbuilder.deviceactionprovider`
-   **Plugin name**: `DeviceAction`
-   **Version**: `1.0`

Manifest entry:

```json
"required_plugins": [
  "widgetbuilder.deviceactionprovider:DeviceAction:1.0"
]
```

Runtime objects:

- Initialization flag: `pluginDeviceactionprovider_initialized`
- Initialization callback map: `pluginDeviceactionproviderEvents = { "onInitialized": onDeviceactionproviderInitialized }`
- Plugin access: `window.plugins.Deviceactionprovider`

## Methods

### `initDevice(deviceId)`

Subscribes to dial and key action events for the specified device. Call this once during
plugin initialization, passing the `device.deviceId` value provided by iCUE.

Subsequent calls for the same provider instance are ignored.

| Parameter  | Type     | Description                                   |
| ---------- | -------- | --------------------------------------------- |
| `deviceId` | `string` | Device identifier provided by [`device.deviceId`](../device-object.md) |

## Signals

### `dialTriggered(actionType, dialIndex)`

Emitted when a dial or key on the device produces an action.

| Parameter    | Type     | Description                          |
| ------------ | -------- | ------------------------------------ |
| `actionType` | `string` | Type of action (see table below)     |
| `dialIndex`  | `int`    | Zero-based index of the dial or key  |

:::note
This signal is emitted only on a real device. It is not emitted in preview mode.
:::

#### Action types

| Value            | Description                      |
| ---------------- | -------------------------------- |
| `"press"`        | Dial or key pressed or released  |
| `"long-press"`   | Dial or key held down            |

## Example

`manifest.json`

```json
"required_plugins": [
  "widgetbuilder.deviceactionprovider:DeviceAction:1.0"
]
```

`index.html`

```javascript
function onDialTriggered(actionType, dialIndex) {
    console.log("Action:", actionType, "Dial:", dialIndex);
}

pluginDeviceactionproviderEvents = {
    "onInitialized": function () {
        if (!window.plugins || !window.plugins.Deviceactionprovider) {
            return;
        }

        window.plugins.Deviceactionprovider.initDevice(device.deviceId);

        window.plugins.Deviceactionprovider.dialTriggered.connect(onDialTriggered);
    }
};

if (typeof pluginDeviceactionprovider_initialized !== "undefined" && pluginDeviceactionprovider_initialized) {
    pluginDeviceactionproviderEvents.onInitialized();
}
```
