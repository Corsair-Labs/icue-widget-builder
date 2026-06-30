
# Device Object


The `device` global object provides information about the device the widget is currently
displayed on. Automatically available in the JavaScript context of every widget.

## Properties

| Property   | Type     | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| `deviceId` | `string` | Unique identifier of the device displaying the widget (UUID without braces) |

```javascript
console.log(device.deviceId); // "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

## Usage

The `device` object is used when a widget or plugin needs to identify the specific device
it is running on — for example, to subscribe to device-specific events.

Pass `device.deviceId` to plugin methods that require a device identifier:

```javascript
pluginDeviceactionproviderEvents = {
    "onInitialized": function () {
        window.plugins.Deviceactionprovider.initDevice(device.deviceId);
        window.plugins.Deviceactionprovider.dialTriggered.connect(onDialTriggered);
    }
};
```

## Availability

The `device` object is injected by iCUE before the widget's scripts execute. It is safe to
access `device.deviceId` at any point after the page loads, including before
`onICUEInitialized` fires.
