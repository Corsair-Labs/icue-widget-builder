---
name: icue-widget-builder
description: Builds iCUE HTML widgets for CORSAIR device screens using the bundled documentation as the source of truth. Use this whenever the user asks to create, modify, debug, review, validate, or package an iCUE widget for Xeneon Edge, Pump LCD, or keyboard LCD.
---

# Skill: iCUE Widget Builder

Use the bundled documentation and references in this skill folder as the source of truth for iCUE widget work.

When the user asks to create an iCUE widget, follow this workflow.

## Documentation Authority

Treat the technical documentation in `docs/` as the primary source of truth, and use `references/` for implementation templates, checklists, and skill-specific workflow guidance.

Use `docs/` first for:

- `manifest.json` requirements
- widget/meta tags
- property types and attributes
- plugin usage
- iCUE globals (`iCUE`, `device`, `iCUE_initialized`)
- packaging layout
- translation behavior
- general guidelines

If older skill guidance, templates, or local references conflict with the updated docs, follow `docs/`.

Do not invent unsupported meta properties, undocumented package layouts, or internal installation-path assumptions.

Before relying on the Widget Builder CLI, verify whether `icuewidget` is actually installed. When shell access is available, check with `icuewidget --version` (or `which icuewidget` as a fallback). If the command is missing or fails, tell the user the CLI does not appear to be installed yet and prompt them to install it before you suggest CLI-based steps.

If the local `icuewidget` CLI is available, prefer using it for `init`, `validate`, and `package` workflows. If it is not available, explain the equivalent manual structure and clearly label the CLI steps as optional until installation is complete.

## Phase 1: Requirements Gathering (MANDATORY — do NOT skip)

**STOP. You MUST ask the user clarification questions before writing any code or designing the widget.** Do not assume defaults or infer answers — ask explicitly and wait for the user's response. Present your best guesses as suggestions the user can accept or modify, but always wait for confirmation.

For a **new widget** or a **major redesign**, ask about ALL of the following:

1. **Widget purpose** — What data should the widget show? What is the source (system sensor, external API, static content, etc.)? If an API is involved, does the user have a preferred one?
2. **Target devices** — Which devices should the widget support? (`dashboard_lcd`, `pump_lcd`, `keyboard_lcd`) For Xeneon Edge, which sizes? (S / M / L / XL, horizontal and/or vertical)
3. **Update interval** — How often should data refresh? What should happen when the widget is offline or the data source is unavailable?
4. **Customization** — What should be configurable by the user in iCUE settings? Suggest sensible defaults but confirm.
5. **Visual style** — Any specific look, theme, or inspiration? Minimalist, detailed, playful, etc.?
6. **Marketplace intent** — Is this intended for Marketplace submission, private use, or both?
7. **iCUE Tools & Plugins** — Ask which common tools and plugins are needed. Use the question template in `references/common-tools.md` → *Phase 1 Questions* section. Ask ALL of the following in one grouped question:
   - Background image/video with editor controls? → MediaViewer
   - Scrolling text ticker? → TickerTracker
   - Locale-aware date formatting? → DateFormatter
   - rgba() from hex color picks? → ColorTools / hexToRGB
   - Hardware sensor values (CPU/GPU/RAM/etc.)? → Sensors plugin + SimpleSensorApiWrapper
   - Current FPS, FPS availability, or active process name? → FPS plugin + SimpleFpsApiWrapper
   - Now-playing song/artist? → SimpleMediaApiWrapper
   - Links that open in the system browser (not inside the widget)? → LinkProvider
   - Stream Deck virtual-device integration? → StreamDeck plugin
   - Dial/key action events from the physical device? → DeviceAction plugin + `device.deviceId`
8. **Other setup dependencies** — API keys, account login, local media assets, or other setup?
9. **Failure behavior** — What should users see in loading, empty, offline, and error states?
10. **Documentation output** — Should the final deliverable include setup notes, compatibility notes, or Marketplace-facing disclosures?

**Do not proceed to Phase 2 until the user has answered these questions.** If the reply is vague or incomplete, ask follow-up questions until you have enough detail to design the widget confidently.

### Fast path for small edits (allowed)

For narrow tasks (for example: rename a property, tweak one control, adjust a single state layout, fix one plugin binding), use a shorter clarification set instead of the full discovery interview.

Minimum fast-path questions:

1. Which files/sections should be changed?
2. What exact behavior should change, and what must stay unchanged?
3. Which devices/sizes must still be supported after the edit?
4. Should existing property names/IDs remain backward compatible?
5. Any setup/plugin/API implications?

If those answers are clear, proceed directly to implementation and verification.

## Phase 2: Research & Design

### 2.1 Reference Documentation

Read only what you need for the current widget:

| File | Read when... |
|------|-------------|
| `docs/index.md` | Starting from the official widget overview, CLI flow, or first-widget example |
| `docs/specification.md` | Checking required files, `manifest.json` schema, supported devices, events, globals, and plugin declaration syntax |
| `docs/icue-global-object.md` | Using `iCUE.iCUELanguage`, `iCUE.fpsLimit`, `iCUE.isPreview`, or `iCUE.defaultTemperatureUnit()` |
| `docs/device-object.md` | Using the injected `device.deviceId` global, especially for DeviceAction setup |
| `docs/controls/index.md` and `docs/controls/*.md` | Choosing official control types and attributes |
| `docs/plugins/index.md` and `docs/plugins/*.md` | Declaring and integrating Sensors, Media, Link, StreamDeck, FPS, and DeviceAction plugins |
| `docs/common-tools.md` | Using bundled common tools and wrappers from local `common/` paths |
| `docs/local-storage.md` | Persisting widget-specific data with `localStorage` keyed by `uniqueId` |
| `docs/translations.md` | Official translation JSON behavior and `tr()` usage |
| `docs/javascript-expressions.md` | Dynamic defaults, values, and module-backed controls |

Skill-specific implementation references:

| File | Read when... |
|------|-------------|
| `references/widget-creation.md` | First time building a widget; package structure, manifest shape, and basic workflow |
| `references/widget-meta-parameters.md` | Choosing property types, settings panel structure, device restrictions, sensor properties, personalization |
| `references/javascript-expressions.md` | Widget needs dynamic defaults, computed values, or module integration |
| `references/local-storage.md` | Widget needs to persist data between sessions |
| `references/html-template.md` | Starting implementation from a full HTML boilerplate |
| `references/css-template.md` | Starting implementation from a base stylesheet |
| `references/lifecycle-and-plugins.md` | Handling iCUE initialization, property access, plugin naming, translation runtime wiring |
| `references/translations.md` | Using `tr()` correctly and generating translation JSON |
| `references/media-backgrounds.md` | Implementing `media-selector`, media backgrounds, blur/brightness, and fallback behavior |
| `references/common-tools.md` | Skill-specific snippets for iCUE tools and plugin wrappers; verify details against `docs/common-tools.md` and `docs/plugins/*.md` |
| `references/security-and-testing-checklists.md` | Running the preflight, security, accessibility, and browser-test checklist |

### 2.2 Device Specifications

| Device | ID | Resolution | Notes |
|--------|-----|-----------|-------|
| Xeneon Edge | `dashboard_lcd` | See size table below | Horizontal & vertical orientations, touch support |
| Pump LCD | `pump_lcd` | 480x480 | Circular display, no touch |
| Keyboard LCD | `keyboard_lcd` | 320x170 | Small screen, no touch |

**Xeneon Edge size slots:**

| Size | Horizontal | Vertical |
|------|-----------|----------|
| Small | 840x344 | 696x416 |
| Medium | 840x696 | 696x840 |
| Large | 1688x696 | 696x1688 |
| Extra Large | 2536x696 | 696x2536 |

### 2.3 Layout Principles

The layout should respond to the shape of the available slot while staying compositionally stable across device and preview surfaces. Follow these rules.

**Rule 1 — Use a single baseline variable for sizing**
Drive fonts, gaps, padding, radii, tile sizes, and badges from one CSS variable such as `--layout-unit` or `--widget-base-unit`. Component styles should consume semantic tokens derived from that baseline, not raw viewport units directly.

Required pattern:

```css
:root {
  --layout-unit: 1vmin;

  /* Named baselines for modes that need stable sizing */
  --device-vertical-s-unit: 4.16px; /* 416px / 100 */
  --preview-vertical-s-unit: calc(100vw * 416 / 69600);

  /* Semantic tokens */
  --space-gap: calc(var(--layout-unit) * 4);
  --space-pad: calc(var(--layout-unit) * 6);
  --font-hero: calc(var(--layout-unit) * 22);
  --font-secondary: calc(var(--layout-unit) * 7);
  --font-label: calc(var(--layout-unit) * 3.5);
  --radius-tile: calc(var(--layout-unit) * 3.5);
}
```

**Rule 2 — Never use raw `vmin`/`vw`/`vh` directly in component rules**
Raw viewport units are acceptable only when defining a baseline variable in `:root` or in a media query override. Do not scatter direct `22vmin`, `6vmin`, or `3.5vmin` values across component selectors, because preview and device surfaces can produce different `vmin` behavior.

**Rule 3 — Use CSS aspect-ratio media queries for layout changes**
Do not write a `detectLayout()` JS function or add `size-s orient-h` body classes for standard layout switching. Use `@media (aspect-ratio)` rules directly in CSS. Use media queries to change layout composition and to swap baselines when required.

**Rule 4 — Treat preview and device as separate rendering modes**
Preview webviews are not guaranteed to match real-device viewport math. When preview differs from device behavior, introduce an explicit preview baseline override rather than patching individual component sizes.

**Rule 5 — For Xeneon Edge vertical layouts, treat Small vertical as the canonical baseline**
Unless the user explicitly requests size-based scaling, vertical Xeneon Edge widgets should use S-vertical (`696x416`) as the design baseline. Larger vertical slots should usually preserve the S-based typography/component proportions and use extra height for centering, spacing, or extra content density — not automatic text growth.

**Rule 6 — Hide, don't shrink**
When a screen is too small to show everything, `display: none` secondary elements. Never try to squeeze everything in. The hero element (big number, key stat) always survives every screen size.

**Rule 7 — One dominant hero element, always visible**
Every widget must have one large centered value — the thing the user reads in under 2 seconds. Size it from semantic tokens (for example `var(--font-hero)`), use `font-weight: 700`, `font-variant-numeric: tabular-nums`, and `white-space: nowrap`.

**Rule 8 — CSS variables for all runtime values**
JS only sets documented runtime variables (for example `--text-color`, `--accent-color`, `--bg-color`, `--widget-opacity`, animation-driving variables, or carefully controlled layout tokens when a hybrid runtime sizing model is necessary). CSS does all rendering. No ad-hoc inline `style.fontSize` / `style.padding` writes per element.

**Rule 9 — Flexbox for layout, absolute only for overlays**
Use `display: flex` for macro layout. Reserve `position: absolute` for elements that overlay (centered hero numbers, needle indicators, footer text on a circular gauge).

**Rule 10 — Check computed styles before populating hidden elements**
When a resize changes which metrics are visible, call `getComputedStyle(el).display !== 'none'` before populating data into hidden DOM nodes.

**Rule 11 — Opacity hierarchy for visual weight**
Primary data: full opacity. Supporting text: 0.7–0.85. Metadata/labels: 0.5–0.6. Creates natural reading hierarchy without extra font sizes.

**Rule 12 — Soft borders should derive from layout tokens**
Use semantic radius tokens such as `var(--radius-tile)`, which are themselves derived from the active baseline.

**Rule 13 — Declare all intended devices in `manifest.json`**
Only list `dashboard_lcd`, `pump_lcd`, `keyboard_lcd` for device types you have actually designed breakpoints and tested layouts for.

Safe-area padding starting points (expressed through the baseline system, not hardcoded per component):

- Balanced / square (pump): about `5 * --layout-unit`
- Wide landscape (dashboard): about `5 * --layout-unit` vertical and `8 * --layout-unit` horizontal
- Tall portrait: about `8 * --layout-unit` vertical and `10 * --layout-unit` horizontal

### 2.4 UX Guidelines

- **Glanceability:** primary information should be understandable in under two seconds
- **States:** implement clear loading, content, empty, and error/offline states
- **Offline resilience:** if the widget depends on network data, prefer showing the last known data with an offline indicator instead of blanking the UI
- **Color defaults:** default to white text on black background unless the request strongly suggests otherwise
- **Settings UI:** keep controls concise, use the correct property type, and never add a manual Save button

### 2.5 Design Specification Before Coding

Before implementing, outline:

```text
Widget Name: [Name]
Target Devices: [dashboard_lcd | pump_lcd | keyboard_lcd]
Target Sizes (Xeneon Edge): [S / M / L / XL, horizontal / vertical]

Properties:
  - [propertyName]: [type] - [description] - [default]

Property Groups:
  - [Widget Name]: [feature properties...]
  - Widget Personalization: [textColor, accentColor, backgroundColor, backgroundMedia, glassBlur, transparency, ...other]

States Required:
  - Loading state
  - Empty state (if applicable)
  - Error/offline state (if API-driven)
```

The personalization group must always follow this order (omit entries the widget doesn't use):

1. `textColor`
2. `accentColor`
3. `backgroundColor`
4. `backgroundMedia`
5. `bgBrightness` *(only when brightness control is implemented)*
6. `glassBlur`
7. `transparency`
8. …any additional widget-specific personalization properties

## Phase 3: Implementation

### 3.1 Package Structure

Generate the package structure documented in the local docs/references. Include required files for the requested scenario and add optional files only when they provide clear value.

Typical files include:

1. `index.html`
2. `manifest.json`
3. `translation.json` when localized/user-facing labels are used
4. widget runtime assets (`images/`, `resources/`, `styles/`, `modules/` as needed)
5. Marketplace assets (`icon.png`, `icon@2x.png`) when Marketplace output is requested

Packaging safeguards:

- prefer documented relative paths
- distinguish runtime widget assets from Marketplace assets
- do not assume internal install directories

If the `icuewidget` CLI is available, prefer:

- `icuewidget init`
- `icuewidget validate`
- `icuewidget package`

If it is not available, say so explicitly and prompt the user to install the CLI before depending on these commands.

### 3.2 Implementation References

Use the references instead of embedding long technical recipes in the main response:

- For starter HTML/JS wiring: `references/html-template.md`
- For base CSS and typography: `references/css-template.md`
- For property types and groups: `references/widget-meta-parameters.md`
- For lifecycle, property access, plugin naming, and translation runtime: `references/lifecycle-and-plugins.md`
- For localization and `tr()` coverage: `references/translations.md`
- For media backgrounds: `references/media-backgrounds.md`
- For validation, accessibility, security, and browser testing: `references/security-and-testing-checklists.md`

### 3.3 Properties and Personalization

Follow the official docs/references exactly for property names, property types, `data-*` attributes, and grouping.

**Personalization properties must always appear in this order** — both in the `<meta>` declarations and in the `x-icue-groups` JSON array. Omit properties the widget doesn't use; never reorder the ones that are present.

| # | Property | Label | Notes |
|---|----------|-------|-------|
| 1 | `textColor` | `tr('Text Color')` | Always include |
| 2 | `accentColor` | `tr('Accent Color')` | Always include |
| 3 | `backgroundColor` | `tr('Background Color')` | Always include; label is "Background Color", not "Background" |
| 4 | `backgroundMedia` | `tr('Background Image')` | Use for new Xeneon Edge widgets so Custom Style clears media correctly |
| 5 | `bgBrightness` | `tr('Background Brightness')` | Include only when brightness control is implemented; label is "Background Brightness", not "Brightness" |
| 6 | `glassBlur` | `tr('Glass Blur')` | Include when media background is implemented |
| 7 | `transparency` | `tr('Background Transparency')` | Always include; label is "Background Transparency", not "Transparency" |
| 8+ | *(widget-specific)* | — | Any additional personalization params go after `transparency` |

Place the personalization group last in `x-icue-groups`.

### 3.4 Quick Reference — Commonly Overlooked Types and Features

- Manifest plugin declarations use `namespace:Name:version` strings in `required_plugins`, for example `widgetbuilder.sensorsdataprovider:Sensors:1.0`.
- Supported plugins in the updated docs: Sensors (`Sensors`), Media (`Media`), Link (`Url`), Stream Deck (`StreamDeck`), FPS (`Fps`), and Device Action (`DeviceAction`). Read `docs/plugins/index.md` and the specific plugin doc before using any plugin.
- Runtime plugin objects live under `window.plugins` with provider-specific names, for example `window.plugins.Sensorsdataprovider`, `Mediadataprovider`, `Linkprovider`, `Streamdeck`, `Fpsdataprovider`, and `Deviceactionprovider`. Check the matching `plugin{Provider}_initialized` flag and `plugin{Provider}Events` map.
- `sensors-combobox` — requires Sensors plugin (`widgetbuilder.sensorsdataprovider:Sensors:1.0` in `manifest.json`). Returns a sensor ID string. Use `SimpleSensorApiWrapper` to fetch sensor data asynchronously. Use `plugins.Sensorsdataprovider.getDefaultSensorIdBlock('temperature')` as `data-default` when appropriate.
- `sensors-factory` — requires Sensors plugin. Returns a `[{sensorId, color}]` array for multi-sensor charts/graphs.
- `search-combobox` — searchable dropdown backed by a module function. Use `data-values="Module.searchFn"` and `data-default="Module.getDefault"`.
- `slider` supports `data-unit-label` (for example `data-unit-label="'%'"`) to show units next to the value.
- `media-selector` — returns an object with `pathToAsset`, `baseWidth`, `baseHeight`, `scale`, `positionX`, `positionY`, and `angle`. Use the full media object, not just the file path. **Note:** iCUE sends `baseWidth`/`baseHeight` (confirmed from official Weather widget source and live testing). Always pass these to `loadMedia()` under the same names.
- `combobox` and `tab-buttons` support key-value format: `[{"key":"left","value":tr('Left')}]`. The key is stored; the value is displayed.
- `iCUE.isPreview` is `true` in preview/mimic mode and `false` on a real device. Use it only for preview-specific behavior, not as a substitute for responsive CSS.
- The injected `device.deviceId` is available for device-specific plugin calls such as `DeviceActionprovider.initDevice(device.deviceId)`.
- FPS data is provided by the FPS plugin (`widgetbuilder.fpsdataprovider:Fps:1.0`), not by the Sensors plugin. Use `SimpleFpsApiWrapper` for promise-based access when available.
- Device Action events are real-device events and may not fire in preview mode. Always provide a non-interactive fallback state.

### 3.5 Media Background Notes

For any widget that supports `media-selector`, use `references/media-backgrounds.md` as the authoritative source.

That reference owns:

- MediaViewer include/path guidance
- full media object fields (`pathToAsset`, `baseWidth`, `baseHeight`, `scale`, `positionX`, `positionY`, `angle`)
- normalization before `loadMedia()`
- background layer model and optional `.main-glass` guidance

The HTML and CSS template references should be treated as examples, not normative rules, for media-background behavior.

### 3.6 External APIs

When using an external API:

- confirm the API choice with the user first
- explain what data will be sent and why
- ask for explicit approval before making any test request
- verify the actual endpoint with a real test request after approval
- confirm the response contains the fields the widget needs
- use HTTPS only
- minimize data sent and cache responses where appropriate
- make refresh rate configurable when polling is required

If no suitable free API exists, ask the user to provide their own API key via a `textfield` property.

## Phase 4: Verification

### 4.1 Browser Testing

Open the widget in a browser and test the target device resolutions.

**Required size checks** — resize the browser to each of these and verify nothing clips or overflows:

| Size | Dimensions | What to check |
|------|-----------|---------------|
| Pump LCD | 480×480 | Hero visible, secondary elements hidden per design |
| Keyboard LCD | 320×170 | Only hero + label showing |
| Dashboard S-H | 840×344 | Short layout — verify correct elements hidden |
| Dashboard S-V | 696×416 | Taller than S-H — verify correct elements showing |
| Dashboard M-H | 840×696 | Full layout, all elements visible |
| Dashboard M-V | 696×840 | Portrait layout |
| Dashboard L-H | 1688×696 | Wide — confirm vmin sizes look proportionally large |
| Dashboard XL-H | 2536×696 | Very wide — all elements readable |

**Use `preview_inspect` to verify vmin is working**, not just screenshots:
```js
// Confirm font-size is vmin-based (not a fixed px value)
getComputedStyle(document.querySelector('.hero')).fontSize
// Should return e.g. "153.12px" at 696px viewport (= 22vmin)
```

At minimum also verify:

- the widget renders outside iCUE for development
- CSS aspect-ratio media queries are triggering (no JS class on body for standard layout switching)
- loading/empty/error/content states behave correctly
- text never clips or overflows at any tested size
- personalization updates apply correctly
- preview rendering stays compositionally consistent with real-device rendering for the same declared mode
- larger vertical Xeneon Edge previews do not inflate secondary text relative to S-vertical unless that scaling is explicitly intended

### 4.2 Preflight Check

Before final output, run the checklist in `references/security-and-testing-checklists.md`.

Pay special attention to:

- documented meta tags only
- correct device IDs and property types
- valid grouping JSON
- translation key coverage
- correct `manifest.json` fields, including `min_app_version`, required OS declarations, `supported_devices`, and `required_plugins` where applicable
- correct usage of `iCUE`, `device`, and plugin globals from the updated docs
- accessibility and contrast
- media background rules if `media-selector` is used
- no unsafe or undisclosed outbound behavior

Before final delivery, verify whether `icuewidget` is installed. If it is, run `icuewidget validate` or explicitly recommend that the user run it. If it is not installed, say that validation via CLI could not be performed and prompt the user to install the CLI first.

## Phase 5: Final Response

Return:

1. the generated files/code
2. a short explanation of the design and supported devices
3. configurable properties and defaults
4. handled states (loading, empty, error/offline, content)
5. setup notes, plugin/API requirements, and Marketplace notes when relevant
6. a reminder to validate/package with the Widget Builder CLI when appropriate, or an install prompt if the CLI is missing

Use a concise summary such as:

```markdown
## Widget: [Name]

### Description
[1-2 sentence description]

### Supported Devices
| Device | Resolution |
|--------|------------|
| [device] | [resolution] |

### Configurable Properties
| Property | Type | Description | Default |
|----------|------|-------------|---------|
| [name] | [type] | [description] | [default] |

### States Handled
- Loading: [description]
- Empty: [description]
- Error/Offline: [description]
- Content: [description]

### Files Generated
| File | Path |
|------|------|
| Widget | `index.html` |
| Stylesheet | `styles/[WidgetName].css` |
| Translation | `translation.json` |
| Icon | `images/[widgetname].svg` |
```
