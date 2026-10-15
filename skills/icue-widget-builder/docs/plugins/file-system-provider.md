# File System Provider

File and directory watching for iCUE widgets. Monitors the local file system and notifies the widget when content changes.

## Overview

-   **Module name**: `widgetbuilder.filesystemprovider`
-   **Plugin name**: `FileSystem`
-   **Version**: `1.0`

Manifest entry:

```json
"required_plugins": [
    "widgetbuilder.filesystemprovider:FileSystem:1.0"
]
```

The File System Provider watches files and directories on the local file system and notifies the widget when content
changes. All paths are checked against the widget's declared [permissions](../permissions.md) before monitoring begins.

## Methods

### `watchPath(path)`

Starts watching a file or directory. If `path` points to a file, `fileChanged` and `fileContentDiff` are emitted on
content changes. If `path` points to a directory, `fileAdded` and `fileRemoved` are emitted when files are created or
deleted inside it.

The path is resolved through the [path macro table](#path-macros) before permission checking occurs.

| Parameter | Type      | Description                                     |
|-----------|-----------|-------------------------------------------------|
| `path`    | `string`  | Absolute path or path with a `{Macro}` prefix   |

### `unwatchPath(path)`

Stops watching the given path. The same macro resolution is applied as in `watchPath`.

| Parameter | Type      | Description                            |
|-----------|-----------|----------------------------------------|
| `path`    | `string`  | Path previously passed to `watchPath`  |

### `clearPaths()`

Stops watching all currently watched paths and clears all internal snapshots.

## Signals

### `fileChanged(path, content)`

Emitted every time a watched file's content changes.

| Parameter  | Type      | Description                       |
|------------|-----------|-----------------------------------|
| `path`     | `string`  | Absolute path of the changed file |
| `content`  | `string`  | Full new content of the file      |

### `fileContentDiff(path, addedLines, removedLines)`

Emitted together with `fileChanged` when the content difference is non-empty. Not emitted if the file was overwritten
with identical content.

| Parameter        | Type        | Description                                                       |
|------------------|-------------|-------------------------------------------------------------------|
| `path`           | `string`    | Absolute path of the changed file                                 |
| `addedLines`     | `string[]`  | Lines present in the new content but not in the previous content  |
| `removedLines`   | `string[]`  | Lines present in the previous content but not in the new content  |

### `fileAdded(path, content)`

Emitted when a new file appears inside a watched directory.

| Parameter  | Type      | Description                   |
|------------|-----------|-------------------------------|
| `path`     | `string`  | Absolute path of the new file |
| `content`  | `string`  | Full content of the new file  |

### `fileRemoved(path)`

Emitted when a file is deleted from a watched directory.

| Parameter | Type       | Description                        |
|-----------|------------|------------------------------------|
| `path`    | `string`   | Absolute path of the removed file  |

## Path Macros

Paths passed to `watchPath` and `unwatchPath` support `{MacroName}` placeholders. The macro is replaced with the
corresponding system path before the path is used.

| Macro         | macOS                                     | Windows                                     |
|---------------|-------------------------------------------|---------------------------------------------|
| `{Home}`      | `~`                                       | `C:/Users/<USER>`                           |
| `{Desktop}`   | `~/Desktop`                               | `C:/Users/<USER>/Desktop`                   |
| `{Documents}` | `~/Documents`                             | `C:/Users/<USER>/Documents`                 |
| `{Downloads}` | `~/Downloads`                             | `C:/Users/<USER>/Downloads`                 |
| `{Music}`     | `~/Music`                                 | `C:/Users/<USER>/Music`                     |
| `{Pictures}`  | `~/Pictures`                              | `C:/Users/<USER>/Pictures`                  |
| `{Movies}`    | `~/Movies`                                | `C:/Users/<USER>/Videos`                    |
| `{Temp}`      | OS-assigned temporary directory           | `C:/Users/<USER>/AppData/Local/Temp`        |
| `{AppData}`   | `~/Library/Preferences/com.corsair/CUE5`  | `C:/Users/<USER>/AppData/Local/Corsair/CUE5` |
| `{Runtime}`   | `~/Library/Application Support`           | `C:/Users/<USER>`                            |

> **Note:** Path macros for `watchPath` use **curly braces** `{Macro}`. Permission path macros in `manifest.json` use
> **angle brackets** `<Macro>`. See [Permissions](../permissions.md#path-macros) for the full list.

## Usage Examples

### Watching a file

Watches a single file and reacts to content changes and line-level diffs.

`manifest.json`:

```json
"required_plugins": [
    "widgetbuilder.filesystemprovider:FileSystem:1.0"
],
"permissions": [
    { "type": "file", "path": "<Documents>/icue-test-watch/watch.txt" }
]
```

`index.html`:

```javascript
var pluginReady = false;
var currentPath = "";

function applyPath() {
    if (!pluginReady) { return; }
    var plugin = window.plugins.Filesystemprovider;
    if (currentPath) {
        plugin.unwatchPath(currentPath);
    }
    currentPath = "{Documents}/icue-test-watch/watch.txt"; // Matches the declared file permission
    if (currentPath) {
        plugin.watchPath(currentPath);
    }
}

function onPluginReady() {
    var plugin = window.plugins.Filesystemprovider;

    plugin.fileChanged.connect(function (path, content) {
        document.getElementById("content").textContent = content;
    });

    plugin.fileContentDiff.connect(function (path, addedLines, removedLines) {
        console.log("added:", addedLines, "removed:", removedLines);
    });

    pluginReady = true;
    applyPath();
}

icueEvents = { onDataUpdated: applyPath };

pluginFilesystemproviderEvents = { onInitialized: onPluginReady };

if (typeof pluginFilesystemprovider_initialized !== "undefined" && pluginFilesystemprovider_initialized) {
    onPluginReady();
}
```

---

### Watching a folder

Watches a directory and reacts to files being added, changed, or removed.

`manifest.json`:

```json
"required_plugins": [
    "widgetbuilder.filesystemprovider:FileSystem:1.0"
],
"permissions": [
    { "type": "folder", "path": "<Documents>/icue-test-watch" }
]
```

`index.html`:

```javascript
var pluginReady = false;
var currentPath = "";

function applyPath() {
    if (!pluginReady) { return; }
    var plugin = window.plugins.Filesystemprovider;
    if (currentPath) {
        plugin.unwatchPath(currentPath);
    }
    currentPath = "{Documents}/icue-test-watch"; // Matches the declared folder permission
    if (currentPath) {
        plugin.watchPath(currentPath);
    }
}

function onPluginReady() {
    var plugin = window.plugins.Filesystemprovider;

    plugin.fileAdded.connect(function (path, content) {
        console.log("added:", path, content);
    });

    plugin.fileRemoved.connect(function (path) {
        console.log("removed:", path);
    });

    pluginReady = true;
    applyPath();
}

icueEvents = { onDataUpdated: applyPath };

pluginFilesystemproviderEvents = { onInitialized: onPluginReady };

if (typeof pluginFilesystemprovider_initialized !== "undefined" && pluginFilesystemprovider_initialized) {
    onPluginReady();
}
```

## Required Permissions

To watch a file or directory the widget manifest must declare both the plugin and the corresponding `file` or `folder`
permission. Without the permission entry the path is silently blocked.

See [Permissions](../permissions.md) for the full reference of permission types and path macros.
