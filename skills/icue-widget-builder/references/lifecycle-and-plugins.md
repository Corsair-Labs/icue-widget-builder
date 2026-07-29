# Lifecycle and Plugin Runtime Guide

Use this reference when wiring widget runtime behavior.

## iCUE Event Binding

Use bare assignment for `icueEvents`:

```js
icueEvents = {
  onDataUpdated: onIcueDataUpdated,
  onICUEInitialized: onIcueInitialized
  // onUpdateRequested: onUpdateRequested
};
```

Do **not** add `var`/`let`/`const` in front of `icueEvents`. In some iCUE execution contexts that prevents the runtime bridge from seeing the handlers.

**`icueEvents` must be referenced even in a widget with zero `x-icue-property` controls.** Confirmed via iCUE's own import validator: a widget whose scripts never assign `icueEvents` fails validation with "index.html or scripts do not reference icueEvents" — this happens even for a purely interactive widget (e.g. Stream Deck buttons only, no data to display) that would otherwise have no real use for `onDataUpdated`. Add a minimal but real assignment rather than omitting it:

```js
function onIcueDataUpdated() {}
function onIcueInitialized() { /* optional: nothing to initialize */ }
icueEvents = { onDataUpdated: onIcueDataUpdated, onICUEInitialized: onIcueInitialized };
```

## Safe Property Access

iCUE may inject properties as globals in a sandboxed function context. Use a helper that checks both `window[name]` and the sandbox-local variable path:

```js
function getIcueProperty(name) {
  if (typeof window !== 'undefined' && Object.prototype.hasOwnProperty.call(window, name)) {
    const value = window[name];
    if (value !== undefined && value !== null && value !== '') return value;
  }
  try {
    const value = Function('return typeof ' + name + ' !== "undefined" ? ' + name + ' : undefined')();
    if (value !== undefined && value !== null && value !== '') return value;
  } catch (e) {}
  return undefined;
}
```

Also use `typeof prop !== 'undefined'` checks before direct property access.

## Lifecycle Rules

Generated widgets should:

- initialize safely before iCUE is ready
- handle `onICUEInitialized`
- handle `onDataUpdated`
- initialize plugins independently when needed
- tolerate missing plugin/data state without crashing
- render meaningful loading/empty/error/content states

If the widget depends on both iCUE properties and plugin readiness, do not assume a fixed initialization order unless the docs guarantee one.

## The `iCUE_initialized` Check Is a Race, Not a Fact — Retry Before Trusting a `false` Read

Confirmed empirically (not theoretical): a widget that runs its boot logic immediately at the end of `<body>` and does a single synchronous `typeof iCUE_initialized !== 'undefined' && iCUE_initialized` check will **intermittently** read `false` even when genuinely running inside iCUE — because iCUE's WebEngine injects `iCUE_initialized` (and the `window.plugins.*` objects) on its own schedule, which sometimes lands before this script executes and sometimes doesn't. This produced a widget that connected to Stream Deck on some launches and silently fell back to "browser preview"/mock-data mode on others, with no code change in between — purely a timing coin-flip.

A one-shot check cannot distinguish "genuinely opened in a plain browser" from "inside iCUE, but not injected yet". Do not commit to the non-iCUE fallback branch on the first negative read. Retry for a short grace window first:

```js
var SD_BOOT_RETRY_MS = 100, SD_BOOT_RETRY_MAX = 15; // ~1.5s grace window
var sdBootAttempts = 0;
function bootCheck() {
  var insideIcue = typeof iCUE_initialized !== 'undefined' && iCUE_initialized;
  if (insideIcue) {
    onIcueInitialized();
    // ...call any onInitialized handlers directly if their *_initialized flag is already true...
    return;
  }
  if (sdBootAttempts < SD_BOOT_RETRY_MAX) {
    sdBootAttempts++;
    setTimeout(bootCheck, SD_BOOT_RETRY_MS);
    return;
  }
  // Only now conclude this is really a non-iCUE browser and fall back to mock/preview mode.
}
bootCheck();
```

`icueEvents` and `pluginStreamdeckEvents` must still be assigned unconditionally, outside this retry loop, before it starts — that registration is what lets iCUE's *real* initialization (whenever it actually lands) reach the widget even during the retry window, independent of what the fallback branch ends up doing.

## Initial Browser Render

**Superseded by the retry pattern above for any widget using a plugin (Sensors, Media, Link, StreamDeck, FPS, DeviceAction).** A one-shot check like the snippet below is the exact shape of the bug described above — do not use it as-is if the widget has any plugin dependency:

```js
// Only safe for widgets with NO plugin dependency, where an `onICUEInitialized` that
// fires slightly late is harmless because nothing has a one-shot "onInitialized" signal
// that could be permanently missed.
if (typeof iCUE_initialized !== 'undefined' && iCUE_initialized) {
  onIcueInitialized();
} else {
  onIcueDataUpdated();
}
```

For anything with a plugin, use `bootCheck()` from the section above instead — it still calls `onIcueInitialized()` directly when the flag is already true, but retries instead of assuming "not iCUE" on the first negative read.

## Plugin Naming Model

When using plugins, keep these names distinct:

- manifest plugin identifier
- runtime `window.plugins.*` object name
- plugin event global name
- wrapper/helper names used in examples

Do not mix similar names unless the docs explicitly require them.

## Translation Runtime Notes

`tr('...')` in `<title>` and `data-label` is often correct iCUE syntax. Do not replace it with plain text unless the docs for that field say otherwise.

## Programmatic Refresh

If the widget uses `onUpdateRequested` or another programmatic update path, rate-limit it to no more than 10 updates per second.
