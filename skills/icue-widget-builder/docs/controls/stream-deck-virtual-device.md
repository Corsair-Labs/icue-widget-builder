# stream-deck-virtual-device

Dropdown and button for selecting a virtual Stream Deck device.
Requires the Stream Deck application to be running.

## Output Value

`string` - Selected virtual Stream Deck device

## Requirements

This control requires Stream Deck application to be running

## Example

HTML:

```html
<meta
    name="x-icue-property"
    content="virtualDevice"
    data-label="tr('Virtual Device')"
    data-type="stream-deck-virtual-device"
/>
```

> **Skill note:** a widget that connects to Stream Deck through the plugin also needs the `application` permission for
> `"Stream Deck"` in `manifest.json` — see [Stream Deck Provider](../plugins/stream-deck.md#required-permission) and
> [Permissions](../permissions.md#application).

## Usage in JavaScript

```javascript
console.log(virtualDevice); // "37b6e7f4-77a4-44b6-8aa9-2bb4d6007139"
```

## Complete Widget Example

Manifest:

```json
{
    "author": "Corsair Team",
    "id": "com.corsair.streamdeckvirtualdevicedemo",
    "name": "Stream Deck Virtual Device Demo",
    "description": "Stream Deck Virtual Device Demo",
    "version": "1.0.0",
    "preview_icon": "icon.png",
    "min_framework_version": "1.8.0",
    "os": [
        {
            "platform": "windows"
        }
    ],
    "supported_devices": [
        {
            "type": "dashboard_lcd"
        },
        {
            "type": "pump_lcd"
        },
        {
            "type": "keyboard_lcd"
        }
    ],
    "required_plugins": [
        "widgetbuilder.streamdeck:StreamDeck:1.2"
    ],
    "permissions": [
        { "type": "application", "application": "Stream Deck" }
    ]
}
```

HTML:

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>tr('Stream Deck Virtual Device Demo')</title>
        <link rel="icon" type="image/svg+xml" href="resources/icon.svg" />

        <meta
            name="x-icue-property"
            content="virtualDevice"
            data-label="tr('Virtual Device')"
            data-type="stream-deck-virtual-device"
        />

        <script type="application/json" id="x-icue-groups">
            [{ "title": "'Settings'", "properties": ["virtualDevice"] }]
        </script>
    </head>
    <body
        style="margin:0;height:100vh;display:flex;align-items:center;justify-content:center;background:#1a1a2e;color:#fff;font-family:sans-serif;"
    >
        <div>
            <span id="deviceId" style="font-size:48px;">--</span>
        </div>
        <script>
            icueEvents = { onDataUpdated: update, onICUEInitialized: update };

            function update() {
                document.getElementById("deviceId").textContent = virtualDevice;
            }

            if (iCUE_initialized) update();
        </script>
    </body>
</html>
```
