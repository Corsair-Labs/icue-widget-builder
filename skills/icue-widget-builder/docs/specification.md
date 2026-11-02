
# iCUE Widget Specification

<WidgetVersion showDetails />

This document provides the technical specification for the iCUE Widgets, including the HTML widget structure, manifest schema, and iCUE communication protocol.

## Widget File Structure

### Required Files

- `index.html` – Main HTML entry point
- `manifest.json` – Widget manifest and metadata
- `icon.svg` or `icon.png` – Widget icon

### Directory Structure

```text
mywidget/
├── index.html
├── manifest.json
├── translation.json
├── modules/
│   └── CitySearch.mjs
├── scripts/
│   └── widget.js
├── styles/
│   └── style.css
└── resources/
    ├── icon.png
    └── thumbnail.png
```

## HTML Structure

Widgets consist of four primary sections:

- **Head** – Metadata, [widget controls](#widget-controls), widget name (`<title>`), and icon (`<link rel="icon" />`)
- **Style** – CSS definitions (inline or external file)
- **Body** – Visual elements and DOM structure
- **Script** – Widget logic and event handlers

### Minimal HTML Example

```html
<!doctype html>
<html lang="en">
	<head>
		<title>My Widget</title>
		<link rel="icon" type="image/x-icon" href="resources/icon.png" />
	</head>
	<body>
		<!-- Widget content -->
	</body>
</html>
```

## Manifest Schema

The `manifest.json` file defines widget metadata, system requirements, and marketplace information. All paths in the manifest are relative to the manifest file location.

### Schema Properties

| Property                | Type       | Required | Description                                                                                                                          |
| ----------------------- | ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `author`                | `string`   | Yes      | Widget author name                                                                                                                   |
| `id`                    | `string`   | Yes      | Unique widget identifier in reverse-DNS format (e.g., `com.author.widget`)                                                           |
| `name`                  | `string`   | Yes      | Widget display name for marketplace                                                                                                  |
| `description`           | `string`   | Yes      | Widget description for marketplace                                                                                                   |
| `version`               | `string`   | Yes      | Widget version in semver format (e.g., `1.0.0`)                                                                                      |
| `preview_icon`          | `string`   | Yes      | Path to widget icon file (relative to manifest)                                                                                      |
| `min_framework_version` | `string`   | Yes      | Minimum required iCUE Widget API version (e.g., `1.0.0`)                                                                             |
| `os`                    | `object[]` | Yes      | Array of supported operating systems objects. Each object requires a `platform` property. At the moment only `windows` is supported. |
| `supported_devices`     | `object[]` | Yes      | Array of supported device types (see [Device Support](#device-support))                                                              |
| `interactive`           | `boolean`  | No       | Enable widget click handling on touch devices (default: `false`)                                                                     |
| `required_plugins`      | `string[]` | No       | Array of required [plugins](./plugins/index.md) in `namespace:Name:version` format                                       |
| `modules`               | `string[]` | No       | Array of JavaScript module paths (see [Module Integration](./javascript-expressions.md#module-integration))              |
| `permissions`           | `object[]` | No       | Array of permission declarations for protected resources (see [Permissions](./permissions.md))                                       |
| `min_app_version`       | `string`   | Yes      | Minimum recommended iCUE version (e.g., `5.47`)                                                                                      |

### Example Manifest

```json
{
	"author": "Corsair Team",
	"id": "com.corsair.weatherwidget",
	"name": "Weather Widget",
	"description": "Display current weather and forecast",
	"version": "1.0.0",
	"preview_icon": "resources/icon.png",
	"min_app_version": "5.46",
	"min_framework_version": "1.0.0",
	"os": [
		{
			"platform": "windows"
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
	],
	"interactive": true,
	"required_plugins": [
		"widgetbuilder.sensorsdataprovider:Sensors:1.0",
		"widgetbuilder.mediadataprovider:Media:1.1"
	],
	"modules": [
		"modules/CitySearch.mjs"
	]
}
```

### Device Support

The `supported_devices` array specifies compatible device types and optional feature requirements.

#### Device Types

| Type                 | Description                            | Examples                                                                        |
| -------------------- | --------------------------------------- | ------------------------------------------------------------------------------- |
| `dashboard_lcd`      | Dashboard displays                    | XENEON EDGE                                                                     |
| `keyboard_lcd`       | Keyboards with integrated LCD screens | VANGUARD 96, VANGUARD PRO 96                                                    |
| `pump_lcd`           | AIO cooler pumps with LCD screens     | iCUE LINK XC7 ELITE LCD, iCUE LINK XD5 ELITE LCD                                |
| `pump_dashboard_lcd` | Pump dashboard displays               | iCUE LINK 5 Inch LCD Screen Module (iCUE LINK TITAN II 360 RX LCD), NAUTILUS II LCD (NAUTILUS II RS LCD / iCUE LINK TITAN II ULTRA 360 LX LCD) |

#### Pump Dashboard Displays

Starting with Widget API v1.10.0 in iCUE v5.54+, `pump_dashboard_lcd` targets the 5 inch LCD Screen Module (iCUE LINK TITAN II 360 RX LCD) and NAUTILUS II LCD (NAUTILUS II RS LCD / iCUE LINK TITAN II ULTRA 360 LX LCD) only. New widgets targeting these devices must declare `pump_dashboard_lcd` in `supported_devices` and set `min_framework_version` to `1.10.0` or later. Neither device supports the `sensor-screen` (touchscreen input) feature. Declaring `pump_lcd` alone does not target these devices.

:::note Backward compatibility
For backward compatibility, both `pump_dashboard_lcd` and the legacy `dashboard_lcd` type currently work for these devices. Existing `dashboard_lcd` widgets remain available only when that entry does not require `sensor-screen`. This legacy `dashboard_lcd` compatibility will be removed in future versions; new widgets must use `pump_dashboard_lcd`.

Feature requirements are evaluated independently for each entry. During this compatibility period, a separate `pump_dashboard_lcd` entry without `sensor-screen` makes a widget available on these devices even if its `dashboard_lcd` entry requires `sensor-screen`.
:::

#### Device Features

Optional `features` array restricts widget to devices with specific capabilities.

| Feature         | Description                           |
| --------------- | -------------------------------------- |
| `sensor-screen` | Device must support touchscreen input |

**Example:**

Widget enable only on dashboard display with sensor screen (XENEON EDGE):

```json
"supported_devices": [
    {
        "type": "dashboard_lcd",
        "features": ["sensor-screen"]
    }
]
```

Widget enable on any pump dashboard display (5 inch LCD Screen Module, NAUTILUS II LCD):

```json
"supported_devices": [
    {
        "type": "pump_dashboard_lcd"
    }
]
```

Widget enable on any pump display (iCUE LINK XC7, XD5 ELITE LCD, etc):

```json
"supported_devices": [
    {
        "type": "pump_lcd"
    }
]
```

Widget enable on any keyboard display (VANGUARD 96, VANGUARD PRO 96, etc):

```json
"supported_devices": [
    {
        "type": "keyboard_lcd"
    }
]
```

## HTML Meta Tags

Meta tags in the `<head>` section control widget presentation in iCUE and define user-configurable properties.

### Required Meta Tags

| Element             | Description                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `<title>`           | Widget name displayed in iCUE in widgetselector. Use `tr('Name')` for [translations](./translations.md) |
| `<link rel="icon" />` | Widget icon displayed in iCUE selector. Supported formats: SVG, PNG, ICO                                           |

### Optional Meta Tags

| Meta Tag Name           | Description                                                            |
| ----------------------- | ---------------------------------------------------------------------- |
| `x-icue-widget-group`   | Group name in widget selector in iCUE (for example `tr('Clock Face')`) |
| `x-icue-widget-preview` | Preview image path for widget selector (128×56px PNG)                  |
| `x-icue-info` (`data-type="app-status"`) | **Not in Elgato's public docs** — confirmed only via Corsair's bundled `StreamDeck` widget. Renders a native companion-app connection-status row in the settings panel; see `references/widget-meta-parameters.md`. |

#### Widget Grouping

Widgets in the same `x-icue-widget-group` will be grouped together in the widget selectorand displayed as variants of the same widget. Widgets with identical names are automatically grouped together.

```html
<meta name="x-icue-widget-group" content="tr('Clock Face')" />
```

:::note
The `tr()` function enables [localized group names](./translations.md).
:::

#### Widget Preview

The `x-icue-widget-preview` meta tag specifies a preview image shown in the widget selector before installation. The image should be 128x56 pixels in size and preferably in PNG format. The path to the image is relative to the `index.html` file.

```html
<meta name="x-icue-widget-preview" content="resources/preview.png" />
```

#### Widget Controls

Widget controls are user-configurable parameters defined as `<meta>` tags with `name="x-icue-property"`. Each control becomes a global JavaScript variable in your widget.

```html
<meta name="x-icue-property" content="textColor" data-label="Text Color" data-type="color" data-default="'#FFFFFF'" />
```

:::info
For complete documentation on available control types, attributes, and examples, see [Widget Controls](./controls/index.md).
:::

##### Property Groups

Property groups organize controls into panels in the iCUE. Groups are defined in a JSON array within a `<script>` element with `id="x-icue-groups"`. JavaScript expressions are supported in `title` and `info` fields.

**Group Schema**

```html
<script type="application/json" id="x-icue-groups">
	[
	    {
	        "title": "tr('Settings')",
	        "properties": ["option1", "option2"]
	        "info": "tr('Optional help text for this group')"
	    },
	    {
	        "title": "tr('Appearance')",
	        "properties": ["textColor", "backgroundColor", "backgroundMedia"],
	    }
	]
</script>
```

**Group Properties**

| Property     | Type       | Required | Description                                                                                                                                             |
| ------------ | ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `title`      | `string`   | Yes      | Group heading displayed in settings panel. Supports [JavaScript expressions](./javascript-expressions.md) including `tr()` for translations |
| `properties` | `string[]` | Yes      | Array of meta parameters (`content` values) to include in this group                                                                                    |
| `info`       | `string`   | No       | Optional help text displayed as tooltip. Supports [JavaScript expressions](./javascript-expressions.md) including `tr()`                    |

#### Device-Specific Behavior

**XENEON EDGE Custom Styles:**

On XENEON EDGE devices, property groups containing `textColor`, `accentColor`, or `backgroundColor` automatically display a "Custom Style" toggle switch in the settings panel (second tab only). When disabled, the device's default color scheme is applied instead of custom values.

This behavior is device-specific and only applies to XENEON EDGE.

## iCUE Events

iCUE injects script blocks into the widget's HTML page at runtime. These scripts provide:

- **Global variables** for each meta parameter (e.g., `textColor`, `fontSize`)
- **`iCUE` global object** with utility functions (see [iCUE Global Object](./icue-global-object.md))
- **`iCUE_initialized`** flag indicating API readiness
- **`device` object** with information about the device displaying the widget (see [Device Object](./device-object.md))
- **Plugin objects** in `window.plugins` namespace [more info about plugins](./plugins/index.md)

The injection occurs before the widget's own scripts execute, making all iCUE data immediately available.

**Event handlers:**

Register callbacks via the global `icueEvents` object:

```javascript
icueEvents = {
	onDataUpdated: onIcueDataUpdated,
	onICUEInitialized: onIcueInitialized,
};

function onIcueInitialized() {
	// Called once when iCUE API is ready
}

function onIcueDataUpdated() {
	// Called on property change; properties available as global variables
}

// Handle late script loading
if (iCUE_initialized) {
	onIcueInitialized();
	onIcueDataUpdated();
}
```

| Event               | Description                                                                                      |
| ------------------- | ------------------------------------------------------------------------------------------------ |
| `onICUEInitialized` | Called once when iCUE API and all data are ready. The event comes once when iCUE is ready.       |
| `onDataUpdated`     | Called when any meta parameter value changes. The event occurs every time any parameter changes. |

## Plugin Events

Each loaded plugin provides its own event object. Register plugin callbacks using `plugin{ModuleName}Events` where `{ModuleName}` is the plugin's module name with namespace removed.

**Example:**

```javascript
pluginSensorsdataproviderEvents = {
	onInitialized: onSensorPluginReady,
};

function onSensorPluginReady() {
	window.plugins.Sensorsdataprovider.sensorValueChanged.connect(function (sensorId, value) {
		// Handle sensor value change
	});
}

if (pluginSensorsdataprovider_initialized) {
	onSensorPluginReady();
}
```

:::info
For detailed plugin documentation, see [Plugins](./plugins/index.md).
:::
