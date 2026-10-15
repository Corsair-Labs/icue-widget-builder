# Widget Permissions

Permissions grant a widget access to protected resources such as the local file system, network endpoints, hardware
devices, and system services. They are declared in the `permissions` array in `manifest.json`.

## How Permissions Work

### Pre-declared permissions

Permissions listed in `manifest.json` are shown to the user during widget installation. The user reviews and approves
them before the widget starts for the first time. Pre-declared permissions are the recommended approach because the
user sees exactly what the widget requires upfront.

### Runtime permission requests

If the widget accesses a resource that was not declared in the manifest, iCUE shows the user a permission request
dialog at the moment the access occurs. The user can **Allow** or **Deny** the request.

- If the user **allows** — the resource becomes accessible and the permission is saved for future sessions.
- If the user **denies** — the request is blocked. The widget must handle this gracefully (see [Handling denied permissions](#handling-denied-permissions) below).

### User-managed permissions

The user can **grant or revoke any permission at any time** from the iCUE widget settings. This means a permission
that was previously approved may become unavailable while the widget is running.

**Widgets must handle permission revocation gracefully.** Do not assume a permission remains approved for the
entire session. A network request may fail, a file read may be blocked, or a hardware API may stop responding at
any point if the user revokes the corresponding permission.

## Handling Denied Permissions

When a permission is denied or revoked, the underlying resource access fails silently (the request is blocked at
the network or file-system level). The widget is responsible for detecting these failures and responding
appropriately:

- For `fetch()` and `WebSocket` — catch network errors and show a fallback state.
- For `file` / `folder` — handle cases where the file read returns no data.
- For hardware types (`microphone`, `camera`, `speaker`, `screen`) — check whether the API is available before
  using it, and handle the case where it becomes unavailable mid-session.

```javascript
fetch("https://api.example.com/data")
    .then(response => response.json())
    .then(data => {
        // handle data
    })
    .catch(error => {
        // Permission may have been denied or revoked — show fallback
        console.warn("Access denied or network error:", error);
    });
```

## Declaration

```json title="manifest.json"
{
    "permissions": [
        { "type": "file",   "path": "<Documents>/my-widget/data.txt" },
        { "type": "folder", "path": "<Documents>/my-widget/logs" },
        { "type": "url",    "domain": "api.example.com", "port": 443 }
    ]
}
```

## Permission Types

### `file`

Grants read access to a single file. Required when the widget requests access to the file via fetch and other native JS
methods, as well as for access via iCUE plugins [File System Provider](./plugins/file-system-provider.md) monitors a
specific file.
Path field supports [path macros](#path-macros).

| Field   | Type      | Required | Description                                    |
|---------|-----------|----------|------------------------------------------------|
| `type`  | `string`  | Yes      | `"file"`                                       |
| `path`  | `string`  | Yes      | Absolute path or path with a `<Macro>` prefix  |

```json
{ "type": "file", "path": "<Documents>/my-widget/config.json" }
```

---

### `folder`

Grants read access to a directory and all files within it. Required when the widget requests access to the file via
fetch and other native JS methods, as well as for access via iCUE plugins
[File System Provider](./plugins/file-system-provider.md) monitors a directory for file additions and removals.
A single `folder` permission covers any file nested at any depth inside the declared path.
Path field supports [path macros](#path-macros).

| Field   | Type      | Required | Description                                    |
|---------|-----------|----------|------------------------------------------------|
| `type`  | `string`  | Yes      | `"folder"`                                     |
| `path`  | `string`  | Yes      | Absolute path or path with a `<Macro>` prefix  |

```json
{ "type": "folder", "path": "<Documents>/my-widget/data" }
```

---

### `url`

Grants network access to a specific host. Required for `fetch()`, `WebSocket`, or any other network request targeting
that host. Applies to `http`, `https`, `ws`, and `wss` schemes. The `data:` and `blob:` URL schemes are always blocked.

| Field    | Type      | Required | Description                                                           |
|----------|-----------|----------|-----------------------------------------------------------------------|
| `type`   | `string`  | Yes      | `"url"`                                                               |
| `domain` | `string`  | Yes      | Hostname, IP address (without protocol), or `"any"` for all external hosts |
| `port`   | `number`  | No       | Restrict access to a specific port, the primary need for `localhost`  |

**Subdomain matching:** declaring `"example.com"` also grants access to `api.example.com`, `cdn.example.com`, and
any other subdomain.

**Wildcard (`"any"`):** grants access to all external (non-localhost) hosts without declaring them individually.

```json
{ "type": "url", "domain": "api.example.com" }
```

```json
{ "type": "url", "domain": "api.example.com", "port": 443 }
```

```json
{ "type": "url", "domain": "any" }
```

For localhost, `port` is a required parameter:

```json
{ "type": "url", "domain": "localhost", "port": 8080 }
```

---

### `microphone`

Grants access to the microphone via the Web Audio API.

```json
{ "type": "microphone" }
```

---

### `camera`

Grants access to the camera via the Media Capture API.

```json
{ "type": "camera" }
```

---

### `speaker`

Grants permission to play audio output.

```json
{ "type": "speaker" }
```


---

### `screen`

Grants access to screen capture via the Screen Capture API.

```json
{ "type": "screen" }
```


---

### `clipboard_read_write`

Grants read and write access to the system clipboard.

```json
{ "type": "clipboard_read_write" }
```

---

### `local_fonts_access`

Grants access to locally installed fonts via the Local Font Access API.

```json
{ "type": "local_fonts_access" }
```

---

### `self_signed_certificate`

Allows connecting to a host that uses a self-signed TLS certificate.

| Field    | Type       | Required | Description                                         |
|----------|------------|----------|-----------------------------------------------------|
| `type`   | `string`   | Yes      | `"self_signed_certificate"`                         |
| `host`   | `string`   | Yes      | Hostname that presents the self-signed certificate  |

```json
{ "type": "self_signed_certificate", "host": "localhost" }
```

---

### `download`

Grants permission to download files. The downloaded data is saved to an iCUE-managed path and is accessible to the
widget during its operation.

```json
{ "type": "download" }
```

---

### `application`

Grants a widget access to a named external desktop application through an iCUE plugin. It is not a browser permission
or a JavaScript API; the plugin verifies the permission before it connects to or exchanges data with the external
application.

| Field         | Type     | Required | Description                                      |
|---------------|----------|----------|--------------------------------------------------|
| `type`        | `string` | Yes      | `"application"`                                  |
| `application` | `string` | Yes      | Name of the external application the widget uses |

```json
{ "type": "application", "application": "Stream Deck" }
```

The application name defines the permission scope. Matching is case-insensitive, so `"Stream Deck"` and
`"stream deck"` represent the same application permission. Declare a separate permission for every external
application the widget needs.

Before using the application, its plugin checks this permission. If no matching permission exists, iCUE requests it
at runtime and the plugin must not access the application until the user approves it. Plugins must react to later
permission changes: disconnect or stop access after denial, and retry their connection after approval.

#### Supported Applications

| Application | Manifest `application` value  | Required plugin                           |
|-------------|-------------------------------|-------------------------------------------|
| Stream Deck | `"Stream Deck"`               | `widgetbuilder.streamdeck:StreamDeck:1.2` |

---

## Path Macros

The `file` and `folder` permission types support `<MacroName>` placeholders in the `path` field. The macro is resolved
to the actual system path at runtime.

| Macro         | macOS                                     | Windows                                     |
|---------------|-------------------------------------------|---------------------------------------------|
| `<Home>`      | `~`                                       | `C:/Users/<USER>`                           |
| `<Desktop>`   | `~/Desktop`                               | `C:/Users/<USER>/Desktop`                   |
| `<Documents>` | `~/Documents`                             | `C:/Users/<USER>/Documents`                 |
| `<Downloads>` | `~/Downloads`                             | `C:/Users/<USER>/Downloads`                 |
| `<Music>`     | `~/Music`                                 | `C:/Users/<USER>/Music`                     |
| `<Pictures>`  | `~/Pictures`                              | `C:/Users/<USER>/Pictures`                  |
| `<Movies>`    | `~/Movies`                                | `C:/Users/<USER>/Videos`                    |
| `<Temp>`      | OS-assigned temporary directory           | `C:/Users/<USER>/AppData/Local/Temp`        |
| `<AppData>`   | `~/Library/Preferences/com.corsair/CUE5`  | `C:/Users/<USER>/AppData/Local/Corsair/CUE5`        |
| `<Runtime>`   | `~/Library/Application Support`           | `C:/Users/<USER>`                                   |

> **Note:** Permission path macros use **angle brackets** `<Macro>`. Path macros in
> [File System Provider](./plugins/file-system-provider.md) `watchPath` calls use **curly braces** `{Macro}`, but refer
> to the same set of locations.

**Example using macros:**

```json
"permissions": [
    { "type": "file",   "path": "<Documents>/tracker/state.json" },
    { "type": "folder", "path": "<AppData>/logs" },
    { "type": "file",   "path": "<Temp>/sensor-cache.txt" }
]
```
