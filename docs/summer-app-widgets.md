# Summer App widgets

This document describes widgets supplied by installed Summer Apps. Summer's separate app-independent collection is documented in [`browser-widgets.md`](browser-widgets.md); browser-owned widgets do not use the app manifest or runtime contracts below.

Possible longer-term directions are tracked separately in [`widgets-future-plans.md`](widgets-future-plans.md). That roadmap is not part of the current API contract.

Summer Apps can provide native widgets, sandboxed widgets, or both. Widget metadata lives in the app's `sap.json` manifest.

Packages whose primary role is supplying widget surfaces can describe themselves
with `"type": ["WidgetProvider"]`. Summer uses `SummerApp` to expose launch
affordances such as **Open here**, address-bar suggestions, and the fallback
`sap://<id>` URL. A widget-only provider therefore omits `SummerApp` and is not
presented as a page that can be opened.

`WidgetProvider` remains descriptive metadata rather than a widget contract. It
does not require a public `widgets` entry, and widget loading does not require
the classification; providers may keep some or all runtime-backed widgets
internal.

### Primary widget launch

When an enabled app declares exactly one public widget, Summer treats it as the
app's primary widget and can offer a direct widget-launch action from App Info.
An app with multiple widgets can opt into the same affordance by setting the
top-level `primaryWidget` field to the ID of one of its declared widgets:

```json
{
  "type": ["WidgetProvider"],
  "primaryWidget": "music-player",
  "widgets": [
    {"id": "music-player", "name": "Music player"},
    {"id": "queue", "name": "Up next"}
  ]
}
```

Summer rejects a `primaryWidget` value that is not a valid declared widget ID.
Apps with multiple widgets and no explicit primary widget remain available in
the Widgets collection without an arbitrary direct-launch choice. The role is
not used to infer this action.

## Built-in app source and release output

Built-in apps are authored under `packages/`. The matching directory under
`public/builtin/sap/` is generated runtime output that Vite copies into the
application bundle; it is not the development source.

The aggregate `npm run build:builtin-apps` command discovers immediate package
directories whose `sap.json` sets `"builtin": true`. Each matching package must
define a `release` script that type-checks, builds into `dist/`, and uses the
shared built-in app releaser. This keeps both discovery and output assembly
generic: adding another built-in package does not require app-specific root
scripts.

The media player follows this layout:

```text
packages/media-player/src/             # TypeScript, Vue SFC, HTML, and CSS source
packages/media-player/tsconfig*.json   # backend ESM and Vue type-checking
packages/media-player/vite.widget.config.mts # isolated Vue widget bundle
public/builtin/sap/media-player/       # generated widget release files
scripts/build-builtin-widget-app.mjs   # shared TypeScript, Vue, and Vite build
scripts/release-builtin-app.mjs        # shared dist, manifest, and asset release
```

Run `npm run build:sap -- media-player` to refresh only that release bundle.
Normal `npm run dev`, `npm run build:vue`, and production builds refresh all built-in
app bundles before Vite starts, so stale package files cannot enter `dist`. The
media-player release fails if its backend or Vue TypeScript does not compile.
Tests type-check the backend in memory and verify that the generated package and
public release files are byte-for-byte current. For iterative work,
`npm run dev:media-player` watches the
package and refreshes its public bundle after every successful save; see the
package [`README.md`](../packages/media-player/README.md) for the source map and
verification commands.

## Interaction and ownership model

Widgets are independent surfaces, not children of a permanent widget panel. A newly opened widget floats above the browser viewport. The user can move it freely or drag it to the left or right edge; Summer shows a dock preview and attaches it only when the user releases there. Apps can ask Summer to open or close their own widgets, but cannot choose or override a dock position.

While a widget keeps the browser chrome above the website, the transparent viewport forwards each mouse gesture as one ordered sequence bound to the tab and input backend that received its press. Cancellation releases any forwarded buttons, and switching tabs or losing the Chromium debugger cannot redirect the remaining phases into another page. Mouse Back and Forward complete through that bound tab's navigation history; the native Windows/Linux `app-command` route is deduplicated against the viewport route so one physical side-button click moves exactly one history entry.

Settings contains the widget collection inside the **Apps & Widgets** hub. Like Summer's grouped Passwords & Autofill settings, the hub uses separate, keyboard-accessible tabs for the related areas instead of adding another permanent panel. The Widgets tab lists every widget declared by an enabled app and lets the user decide which ones remain open. Floating widgets normally can be resized directly from either bottom corner with a pointer or the arrow keys. A browser-owned widget may suppress freeform handles when its semantic sizes intentionally select different interfaces; Market Prices uses only the preset resize control because small/medium are the auto-height ticker and large is the full dashboard/configuration. Summer persists custom dimensions, the floating position, and optional left/right dock placement; using the preset resize control moves to the next declared `small`, `medium`, or `large` `sizePresets` dimensions when available.

This separation is intentional:

- The manifest describes what widgets an app offers.
- The app controls when one of its widgets is useful enough to open or close.
- The user controls whether it remains open and where it is placed.
- Summer owns persistence, docking, and the security boundary. It provides the outer header by default, but the app may hide or replace it.

### Widget tags

A widget can opt into Summer's localized catalog filters with a `tags` array. Tags use stable taxonomy IDs rather than display text; Summer owns and translates the labels. Supported IDs are `accessibility`, `communication`, `finance`, `games`, `media`, `productivity`, `sports`, and `system`. Unknown IDs are ignored, and a widget without recognized tags remains visible when no tag filter is active.

```json
{
  "id": "music-player",
  "name": "Music player",
  "tags": ["media"],
  "renderer": {"type": "sandboxed", "entry": "widgets/player/index.html"}
}
```

## Header ownership

Every widget chooses its header mode in `sap.json`:

- `"host"` is the default and keeps Summer's title, drag area, and controls.
- `"hidden"` removes the header when the app wants a minimal surface.
- `"custom"` gives the app the complete widget surface so it can render its own header and controls.

```json
{
  "id": "music-player",
  "name": "Music player",
  "header": "custom",
  "renderer": {
    "type": "sandboxed",
    "entry": "widgets/player/index.html"
  }
}
```

Native widgets can use `hostCommand` on a structured action when their host header is hidden:

```js
{
  type: "actions",
  actions: [
    {id: "refresh", label: "Refresh", hostCommand: "refresh"},
    {id: "settings", label: "Settings", hostCommand: "settings"},
    {id: "close", label: "Close", hostCommand: "close"}
  ]
}
```

Supported commands are `close`, `refresh`, `resize`, `float`, and
`settings`. The settings command opens the declaring app's settings when it
has a settings declaration; the other commands affect only the widget sending
the command.

### Opening and closing from an app

The activation context exposes a controller scoped to the calling app. A widget ID must be declared in that app's manifest; one app cannot control another app's widgets.

```js
let widgets;

export function activate(context) {
  widgets = context.widgets;
}

export function showPlayer() {
  widgets.open("music-player"); // Opens floating unless the user already placed it.
}

export function stopPlayer() {
  widgets.close("music-player");
}

export function deactivate() {
  widgets = undefined;
}
```

Calling `open` for a widget that is already present is idempotent and preserves
the user's size and placement. Calling `close` removes it from its current
floating or docked location. If the user unpins a widget, later app-driven
`open` calls are ignored. An explicit browser UI request whose purpose is to
open that widget overrides the dismissal, just like opening it from Apps &
Widgets; apps cannot mark their own requests as user-initiated.

### Mobile presentation ownership

Reviewed packaged apps can reuse the same sandboxed renderer on Android and
iOS. Set `mobileRenderer` when the phone needs a different entry, or omit it to
reuse `renderer`. A mobile renderer may be `native` or `sandboxed`:

```json
{
  "id": "music-player",
  "name": "Music player",
  "renderer": {"type": "sandboxed", "entry": "widgets/player/index.html"},
  "mobileRenderer": {"type": "sandboxed", "entry": "widgets/player/index.html"}
}
```

On mobile, `widgets.open("music-player", {mode: "expanded", iconUrl})` may
request an expanded overlay and provide a validated HTTPS launcher image. A
host capability may instead issue an opaque `iconToken`; passing that token to
`widgets.open` lets Summer resolve the image without disclosing its URL to the
app.
`mode: "compact"` requests the minimized presentation. The app can later call
`widgets.close` or, from its sandboxed document, call
`bridge.present("compact" | "expanded" | "hidden")`.

The sandboxed document owns the pixels and interactions for its Library tile,
expanded overlay, and long-press menu. The host context identifies
`platform: "android" | "ios"` and `presentation: "tile" | "expanded" |
"menu"`, so one renderer can adapt without a mobile-only Vue implementation.
The document also owns its polling or push cadence; `refreshAfterMs` remains a
native-snapshot facility.

Summer still owns the native overlay mechanics: circle hit-target geometry,
drag bounds and persisted position, safe areas, stacking over the website view,
back/Escape recovery, validated bridge routing, and failure fallback. The app
controls the full-bleed launcher image, but the user's circle/tile preference
may override how a compact request is presented. Apps cannot access raw
Capacitor/native bridges, change another app's widget, or suppress browser and
OS safety controls.

Mobile sandboxed widgets are currently limited to the explicit reviewed bundle
allowlist. The iframe and CSP are defense in depth for those packaged apps, not
an installation security model for downloaded code. Unreviewed apps require a
separate signed installation/revocation design and a native frame-isolation
audit before custom HTML can be enabled.

## Native widgets

Native widgets omit `renderer` (or use `{"type":"native"}`) and export `renderWidget(request)` from the app's main module. The function returns a structured snapshot containing `text`, `metric`, `progress`, `list`, and `actions` blocks. Summer validates that snapshot and owns all resulting UI.

```json
{
  "id": "account-status",
  "name": "Account status",
  "defaultSize": "medium"
}
```

```js
export function renderWidget({widgetId, locale, colorScheme}) {
  return {
    title: "Account status",
    blocks: [{type: "metric", label: "Messages", value: "42"}],
    refreshAfterMs: 30000
  };
}
```

## Sandboxed widgets

Sandboxed widgets point to an HTML entry inside the app directory. Relative scripts, styles, images, fonts, and media are served from the entry file's directory through Summer's private `summer-widget:` protocol.

```json
{
  "id": "music-player",
  "name": "Music player",
  "defaultSize": "medium",
  "renderer": {
    "type": "sandboxed",
    "entry": "widgets/player/index.html",
    "allow": {
      "connect": ["https://api.music.example"],
      "images": ["https://artwork.music.example"],
      "media": ["https://stream.music.example"]
    }
  }
}
```

Origins are denied unless listed. Only exact HTTPS origins are accepted for images and media; connections accept exact HTTPS or WSS origins. Remote servers must still permit the request through their own CORS policy.

Trusted built-in widgets may additionally declare `"tabCapture": true`. This does not grant capture access to the sandbox. Summer obtains the source-bound stream in the top-level widget host and renders it behind the frame; the widget only reports a bounded preview rectangle for cropping. The flag is ignored for third-party apps.

The built-in media player uses that host-owned surface for YouTube. Its provider
relays the current page video element locally, while the sandbox receives only
stream availability and reports layout. If the element relay is unavailable,
the player uses artwork instead of continuously capturing YouTube's full
compositor surface. Other sources may use bounded compositor frames when an
element stream is not required.

The frame uses `sandbox="allow-scripts"`. It receives no Electron or Node bridge, cannot open popups or forms, and cannot navigate away. Its response policy blocks objects, nested frames, workers, undeclared network access, camera, microphone, geolocation, payment, display capture, and fullscreen.

### Host context

The widget announces readiness and receives theme and locale context through `postMessage`:

```js
parent.postMessage({source: "summer-sandboxed-widget", type: "ready"}, "*");

window.addEventListener("message", event => {
  if (event.source !== parent || event.data?.source !== "summer-widget-host") return;
  if (event.data.type === "connect" && event.ports[0]) {
    // Keep this port and use it for subsequent action messages and responses.
    const hostPort = event.ports[0];
    hostPort.start();
  }
  if (event.data.type === "context") {
    const {widgetId, headerMode, locale, colorScheme, theme, viewport, privateWindow} = event.data;
    // Apply the context to the custom UI.
  }
});
```

`privateWindow` is browser-owned context. A widget handling data outside
Summer's private-profile cleanup guarantees should disable those operations
when it is true; widget-provided storage or payload values are not authoritative.

The dedicated `MessagePort` is optional; window messages remain available for
simple widgets and startup. The port is preferable for frequent requests
because it gives the widget an isolated, persistent action channel.

### Calling the app backend

The frame cannot use IPC directly. It sends an action ID and optional JSON payload to the host instead:

```js
parent.postMessage({
  source: "summer-sandboxed-widget",
  type: "action",
  actionId: "play",
  requestId: "play-1",
  payload: {trackId: "track-42"}
}, "*");
```

The app handles that request in its main module:

```js
export async function handleWidgetAction({widgetId, actionId, payload}) {
  if (widgetId === "music-player" && actionId === "play") {
    return {data: {playing: true, trackId: payload.trackId}};
  }
}
```

Summer validates the payload and result as JSON-compatible data, limits each to 16 KB, and sends the result back:

```js
window.addEventListener("message", event => {
  if (event.source !== parent || event.data?.source !== "summer-widget-host") return;
  if (event.data.type === "action-result" && event.data.requestId === "play-1") {
    console.log(event.data.ok, event.data.data, event.data.error);
  }
});
```

An action result may also include a safe `openUrl`, which Summer opens outside the sandbox after validating its protocol.
For visual data, the backend may return
`binaryImage: {mimeType, bytes: Uint8Array}`. Summer accepts JPEG, PNG, and WebP
images up to 128 KB and transfers the underlying buffer to the sandbox without
base64 conversion. The sandbox receives `binaryImage.bytes` as an `ArrayBuffer`.

### Custom header controls

A sandboxed custom header can invoke the same bounded host controls without receiving Electron access:

```js
parent.postMessage({
  source: "summer-sandboxed-widget",
  type: "host-command",
  command: "close"
}, "*");
```

To preserve floating and edge-docking behavior without feedback jitter, a custom drag handle sends its starting frame coordinate followed by screen-coordinate deltas. The app should capture the pointer so it continues receiving movement until release:

```js
const header = document.querySelector("header");
let lastScreenPoint;

function sendDrag(phase, event) {
  const screenPoint = {x: event.screenX, y: event.screenY};
  const start = phase === "start";
  parent.postMessage({
    source: "summer-sandboxed-widget",
    type: "host-drag",
    phase,
    coordinateMode: "delta",
    x: start ? event.clientX : screenPoint.x - lastScreenPoint.x,
    y: start ? event.clientY : screenPoint.y - lastScreenPoint.y
  }, "*");
  lastScreenPoint = phase === "end" ? undefined : screenPoint;
}

header.addEventListener("pointerdown", event => {
  if (event.button !== 0 || event.target.closest("button")) return;
  header.setPointerCapture(event.pointerId);
  sendDrag("start", event);
});
header.addEventListener("pointermove", event => {
  if (header.hasPointerCapture(event.pointerId)) sendDrag("move", event);
});
header.addEventListener("pointerup", event => sendDrag("end", event));
```

The `context` message includes `headerMode`, allowing one renderer to adapt to `host`, `hidden`, or `custom` mode.
