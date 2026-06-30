# media-selector

Media file selector with transformation controls (scale, position, rotation).

**Platform notes:**

-   Windows: supports `AV1`, `VP8`, `VP9` video codecs
-   macOS: video codecs are **not** supported

## Attributes

| Attribute      | Type        | Description                    | Support JS expressions  |
| -------------- | ----------- | ------------------------------ | ----------------------- |
| `data-filters` | `string[]`  | File type filters              | ✅                     |

## Output Value

`object` (or `undefined` if no media selected):

| Property      | Type     | Description               |
| ------------- | -------- | ------------------------- |
| `pathToAsset` | `string` | URL/path to selected file |
| `scale`       | `number` | Scale factor (1.0)        |
| `positionX`   | `number` | X position offset         |
| `positionY`   | `number` | Y position offset         |
| `baseWidth`   | `number` | Original image width      |
| `baseHeight`  | `number` | Original image height     |
| `angle`       | `number` | Rotation angle in degrees |

## Example

```html
<meta
    name="x-icue-property"
    content="backgroundImage"
    data-label="tr('Background Image')"
    data-type="media-selector"
    data-filters="['*.png', '*.jpg', '*.gif']"
/>
```

```javascript
if (typeof backgroundImage !== "undefined") {
    console.log(backgroundImage.pathToAsset); // "file:///path/to/image.png"
    console.log(backgroundImage.scale); // 1.5
    console.log(backgroundImage.angle); // 45
}
```

## MediaViewer Helper

See [MediaViewer documentation](../common-tools.md) for simplified media handling.

## Complete Widget Example

Manifest:

```json
{
    "author": "Corsair Team",
    "id": "com.corsair.mediaselectordemo",
    "name": "Media Selector Demo",
    "description": "Media Selector Demo",
    "version": "1.0.0",
    "preview_icon": "icon.png",
    "min_framework_version": "1.0.0",
    "os": [
        {
            "platform": "windows"
        },
        {
            "platform": "mac"
        }
    ],
    "supported_devices": [
        {
            "type": "dashboard_lcd",
            "features": ["sensor-screen"]
        },
        {
            "type": "pump_lcd"
        },
        {
            "type": "keyboard_lcd"
        }
    ]
}
```

HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>'Media Selector Demo'</title>
    <link rel="icon" type="image/svg+xml" href="resources/icon.svg">

    <meta name="x-icue-property" content="backgroundImage"
          data-label="tr('Background Image')" data-type="media-selector"
          data-filters="['*.png', '*.jpg', '*.gif']">

    <script type="application/json" id="x-icue-groups">
    [{"title": "tr('Settings')", "properties": ["backgroundImage"]}]
    </script>

    <!-- Bundle these files from docs/common into your widget folder -->
    <link rel="stylesheet" href="common/tools/media_viewer/MediaViewer.css">
    <script src="common/tools/media_viewer/MediaViewer.js"></script>
</head>
<body style="margin:0;height:100vh;background:#1a1a2e;color:#fff;font-family:sans-serif;">
    <div id="media" style="position:relative;width:100%;height:100%;"></div>
    <div id="info" style="position:relative;z-index:1;padding:20px;">No media</div>
    <script>
        const viewer = new MediaViewer({ container: document.getElementById("media") });
        icueEvents = { "onDataUpdated": update, "onICUEInitialized": update };

        function update() {
            if (typeof backgroundImage === "undefined") {
                viewer.clear();
                document.getElementById("info").textContent = "No media";
            } else {
                viewer.loadMedia({
                    path: backgroundImage.pathToAsset,
                    baseWidth: backgroundImage.baseWidth,
                    baseHeight: backgroundImage.baseHeight,
                    scale: backgroundImage.scale,
                    positionX: backgroundImage.positionX,
                    positionY: backgroundImage.positionY,
                    angle: backgroundImage.angle
                });
            }
        }
        if (iCUE_initialized) update();
    </script>
</body>
</html>
```

For the example to work, you also need to add icon.svg

Copy `common/tools/media_viewer/` from the documentation bundle into your widget folder, then include the files as shown above.
