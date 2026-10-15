# Widget Permissions

Skill-specific guidance for the manifest `permissions` array. The authoritative reference for permission types, fields,
and path macros is `docs/permissions.md` — read it before declaring anything, and follow it if this file disagrees.

Permissions gate a widget's access to protected resources: external network hosts, local files and folders, hardware
(microphone, camera, speaker, screen), the clipboard, local fonts, downloads, self-signed TLS hosts, and external
desktop applications reached through plugins.

## How permissions behave (what to design around)

- **Pre-declared** permissions in `manifest.json` are shown to the user at install time and approved before the widget
  first starts. This is the recommended path.
- **Undeclared** access triggers a runtime Allow/Deny dialog at the moment the widget touches the resource. Never
  design a widget that relies on this prompt — it appears mid-use, on the device surface, at an unpredictable moment.
- The user can **grant or revoke any permission at any time** from iCUE widget settings, including while the widget is
  running. A permission that worked a minute ago can stop working.
- When access is denied or revoked, the underlying request is **blocked silently** (network or file-system level). The
  widget gets a failed `fetch`, an empty read, or an unavailable API — not an exception that says "permission".

## Feature → permission map

Ask which of these the widget needs during Phase 1, then declare exactly those.

| The widget needs to... | Declare | Notes |
|---|---|---|
| Call an external API (`fetch`, `XMLHttpRequest`, `WebSocket`) | `{ "type": "url", "domain": "api.example.com" }` | One entry per host. Subdomains are covered (`example.com` covers `api.example.com`). `http`, `https`, `ws`, `wss` only; `data:` and `blob:` are always blocked. Optional `port`. |
| Call a service on the local machine | `{ "type": "url", "domain": "localhost", "port": 8080 }` | `port` is **required** for `localhost`. |
| Call user-chosen / arbitrary external hosts | `{ "type": "url", "domain": "any" }` | All external (non-localhost) hosts. Last resort — see least privilege below. |
| Read one local file (native JS or File System Provider) | `{ "type": "file", "path": "<Documents>/my-widget/data.json" }` | Read access to that file. |
| Read/watch a directory and everything beneath it | `{ "type": "folder", "path": "<Documents>/my-widget/data" }` | Covers files at any depth. |
| Use the microphone | `{ "type": "microphone" }` | Web Audio API. |
| Use the camera | `{ "type": "camera" }` | Media Capture API. |
| Play audio | `{ "type": "speaker" }` | Audio output. |
| Capture the screen | `{ "type": "screen" }` | Screen Capture API. |
| Read/write the clipboard | `{ "type": "clipboard_read_write" }` | |
| Enumerate installed fonts | `{ "type": "local_fonts_access" }` | Local Font Access API. |
| Connect to a host with a self-signed certificate | `{ "type": "self_signed_certificate", "host": "localhost" }` | Field is `host` (not `domain`). Network access to that host still needs its own `url` entry. |
| Download files | `{ "type": "download" }` | Data is saved to an iCUE-managed path. |
| Talk to an external desktop app through a plugin | `{ "type": "application", "application": "Stream Deck" }` | Not a browser/JS permission; the plugin checks it. Only `"Stream Deck"` (plugin `widgetbuilder.streamdeck:StreamDeck:1.2`) is documented. |

Plugins and permissions:

- **Stream Deck Provider** → `application` permission for `"Stream Deck"` (see `docs/plugins/stream-deck.md`).
- **File System Provider** (`widgetbuilder.filesystemprovider:FileSystem:1.0`) → a `file` permission to watch a file or a
  `folder` permission to watch a directory. Without it the path is silently blocked.
- Sensors, Media, Link, FPS, Device Action, and Notifications providers have **no documented permission requirement**.
  Do not add permission entries for them.

## Declaration

```json title="manifest.json"
{
  "required_plugins": [
    "widgetbuilder.filesystemprovider:FileSystem:1.0"
  ],
  "permissions": [
    { "type": "url",    "domain": "api.open-meteo.com" },
    { "type": "file",   "path": "<Documents>/my-widget/state.json" },
    { "type": "folder", "path": "<AppData>/logs" }
  ]
}
```

### Path macros — two syntaxes, same locations

| Where | Syntax | Example |
|---|---|---|
| `permissions[].path` in `manifest.json` | **angle brackets** `<Macro>` | `<Documents>/icue-test-watch/watch.txt` |
| `watchPath()` / `unwatchPath()` in File System Provider | **curly braces** `{Macro}` | `{Documents}/icue-test-watch/watch.txt` |

Macros: `Home`, `Desktop`, `Documents`, `Downloads`, `Music`, `Pictures`, `Movies`, `Temp`, `AppData`, `Runtime`
(OS-specific resolution tables are in `docs/permissions.md`). The path passed to `watchPath` must fall inside a declared
`file` / `folder` permission — copy the same path, changing only the bracket style.

## Least privilege

- Prefer a specific hostname over `"any"`. Use `"any"` only when the user genuinely supplies arbitrary endpoints
  (for example a configurable webhook URL) and tell the user why.
- Add `port` to `url` entries when the service uses a fixed port.
- Prefer `file` over `folder` when only one file is needed; use the deepest folder that covers the need.
- Declare nothing the code does not use. Remove permissions when a feature is removed.
- Hardware permissions (`microphone`, `camera`, `screen`, `clipboard_read_write`) are privacy-sensitive: confirm the user
  really wants the feature and disclose it in the setup/Marketplace notes.
- Keep the existing rules: HTTPS only for external APIs, no remote `<script src>`, no remote web fonts. Plain `http`/`ws`
  is acceptable only for a user-controlled `localhost` service, and only when the user asked for it.

## Handling denied or revoked permissions

Because blocking is silent, the widget must detect failure itself and degrade gracefully:

- **`fetch` / `WebSocket`** — catch network errors; show the last known data plus an offline/blocked indicator, never a
  blank widget. Treat a permission block the same as "offline" — they are indistinguishable from the widget's side.
- **`file` / `folder` reads and File System Provider** — handle empty/no data; show an empty or setup state that tells
  the user to allow file access in iCUE widget settings.
- **Hardware APIs** — check API availability before use and handle it vanishing mid-session.
- **Plugins gated by `application`** — expect a disconnect after denial and a reconnect after approval.
- Keep retrying on the normal refresh interval (with backoff) so the widget recovers on its own after the user re-grants
  access. Do not retry in a tight loop.

```javascript
async function refresh() {
  try {
    const response = await fetch(API_URL);
    if (!response.ok) { throw new Error('HTTP ' + response.status); }
    const data = await response.json();
    lastGood = data;
    render(data, { stale: false });
  } catch (err) {
    // Permission may have been denied or revoked, or the network is down — same handling.
    console.warn('Data unavailable (offline, blocked, or permission revoked):', err);
    render(lastGood, { stale: true });  // last known data + offline indicator, or an empty/setup state when null
  }
}
```

## Permission audit (run after every change that touches the widget code)

The Widget CLI does not validate `permissions`, and the browser does not enforce them, so reconcile code and manifest by hand. This is mandatory before final delivery and after any edit that adds, changes, or removes a permission-gated call.

**1. Scan the widget's own files** (`index.html`, `modules/`, `scripts/`; skip bundled `common/` tools unless they were modified) for permission-gated calls. Heuristic patterns, not exhaustive — also read for indirect use (URLs built from variables or settings, wrappers around `fetch`):

| Pattern to search | Permission it implies |
|---|---|
| `fetch(`, `XMLHttpRequest`, `new WebSocket(`, `new EventSource(`, `navigator.sendBeacon(`, absolute `http(s)://` / `ws(s)://` strings | `url` for each distinct host (+ `port` when fixed; `localhost` needs `port`) |
| `watchPath(`, `unwatchPath(` | `file` / `folder` covering each path (`{Macro}` in code ↔ `<Macro>` in manifest) |
| `fetch(` / reads of a local path, `file://` | `file` / `folder` |
| `getUserMedia` (audio) | `microphone` |
| `getUserMedia` (video) | `camera` |
| `getDisplayMedia` | `screen` |
| `<audio`, `new Audio(`, `AudioContext`, `.play()` on media producing sound | `speaker` |
| `navigator.clipboard` | `clipboard_read_write` |
| `queryLocalFonts` | `local_fonts_access` |
| `download` attribute, programmatic file download | `download` |
| TLS to a self-signed host | `self_signed_certificate` (`host`) + `url` |
| `connectStreamDeck(`, `window.plugins.Streamdeck` | `application` → `"Stream Deck"` |

**2. Reconcile in both directions:**

- every hit has a matching manifest entry (right type, host/path, and `port`), and
- every manifest entry is used by some hit — remove unused ones (least privilege).
- confirm no entry was invented for plugins that document no permission (Sensors, Media, Link, FPS, Device Action, Notifications).

**3. Check failure paths:** each gated call has a denied/revoked-access branch (see above).

**4. Report** the outcome in the reply, for example:

```text
Permission audit: added url api.open-meteo.com (new fetch in weather.js); removed folder <Temp>/cache (unused after refactor); no other changes.
```

or `Permission audit: no permission-gated calls; permissions array omitted.`

## Testing permissions

- Browser testing outside iCUE does **not** enforce manifest permissions, so a widget that works in the browser can
  still fail in iCUE if a `permissions` entry is missing or too narrow. Check every host/path the code touches against
  the manifest by hand (grep for `fetch(`, `WebSocket(`, `XMLHttpRequest`, `watchPath(`).
- Exercise the failure branches by simulating a blocked request (offline mode, a deliberately wrong host) and confirm the
  widget shows its fallback state instead of breaking.
- When validating in iCUE, test once with the permission denied/revoked in widget settings and once granted.

## What to tell the user

In the final response list every declared permission with the reason — it mirrors what the user will be asked to approve
at install time:

```markdown
### Permissions
| Permission | Scope | Why |
|---|---|---|
| `url` | `api.open-meteo.com` | Fetch weather data |
| `application` | `Stream Deck` | Connect to the Stream Deck app |
```

Say "none required" explicitly when the widget declares no permissions.
