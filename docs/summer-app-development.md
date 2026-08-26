# Developing Summer Apps

This human guide covers the end-to-end structure of a Summer App. The manifest
schema is [`sap.schema.json`](../sap.schema.json), the current runtime contract
is in
[`electron/main/apps/apploader.ts`](../electron/main/apps/apploader.ts), and
small packages can be found under [`examples`](../examples).

## Choose the package target

Decide how the app will be distributed before creating files:

| Target | Source location | Additional wiring |
| --- | --- | --- |
| Example or locally installed app | `examples/<app-id>/` | None |
| App bundled with Summer | `packages/<app-id>/` | Built-in registry, package release script, generated runtime output, and focused tests |
| Store listing for an external app | The app's own repository or supplied artifact | `public/appregistry.json` only after real download and repository metadata exist |

A Summer App is not a browser-owned `summer://` page. Use
[`summer-builtin-webpages.md`](summer-builtin-webpages.md) for trusted browser
UI.

Choose the roles the runtime actually needs:

- `SummerApp` for an openable `sap://<id>` page with a `serve(request)` export;
- `SuggestionProvider` for `suggest(queryState)` and
  `openSuggestion(parameters)`;
- `WidgetProvider` for an app whose primary surface is a declared widget; and
- `RawTextEngine` only after inspecting its current runtime consumer.

`lifeCycle: "perm"` makes the runtime load the app at browser startup and keep
it active. Omit it unless the app genuinely requires permanent activation.

## Create the minimum package

Every app needs `sap.json` and a runtime entry:

```json
{
  "name": "Example App",
  "id": "example-app",
  "main": "./index.mjs",
  "version": "0.1.0",
  "description": "A concise user-facing description.",
  "icon": "./icon.png",
  "offlineSupport": {
    "available": true,
    "offerOnConnectionError": true
  },
  "platforms": {
    "windows": true,
    "ios": false,
    "macos": true,
    "linux": true,
    "android": false,
    "any": false
  },
  "type": ["SummerApp"],
  "tags": ["summer:apps", "summer:offline"]
}
```

### Opening declared file types

A desktop Summer App can opt into browser-owned file routing with a declarative
manifest field:

```json
{
  "fileHandlers": [{
    "id": "documents",
    "extensions": [".md", ".txt"],
    "intent": "edit"
  }]
}
```

Handler IDs are stable app-local lowercase identifiers. Extensions are
lowercase final extensions including the dot. An app may declare up to 16
handlers and 32 unique extensions per handler. Executable, script, shortcut,
and active-content formats are rejected. `view` grants read access; `edit`
also permits bounded saves. The declaration is valid only for manifests with
the `SummerApp` role and grants no filesystem access by itself.

After the user reviews a dropped file, Summer opens the unique eligible app at
its root and exposes `window.summerFile` only to that exact grant page:

```ts
const file = await window.summerFile.open();
// {name, extension, size, intent, resourceUrl, revision}
const bytes = new Uint8Array(await (await fetch(file.resourceUrl)).arrayBuffer());

await window.summerFile.save({bytes, revision: file.revision}); // edit only
```

The resource URL and revision are opaque capabilities. Do not persist or share
them. Summer rechecks installation, enabled state, platform support, extension,
handler ID, and intent when the grant is used. Apps never receive a native path.
If more than one enabled app claims an extension, Summer does not silently pick
one.

Use a lowercase stable ID accepted by the schema and a SemVer-compatible
version. Do not reuse another app's ID. Keep the package version, manifest
version, registry version, and generated release aligned whenever those copies
exist.

Keep filter categories separate from search discovery. `tags` are small,
app-authored categories shown as filter chips. Manifests and bookmarks explicitly
opt into `summer:apps`, `summer:widgets`, `summer:offline`, `summer:games`,
`summer:accessibility`, or `summer:utilities`; Summer localizes their display
labels but never infers them from roles or capabilities. These tags are filter
metadata, not authorization. Put descriptive search terms such as `video`,
`subtitles`, `terminal`, or provider names in `keywords` instead. Keywords help
find an app but never become filter chips.

Every new app should declare all six `platforms` booleans: `windows`, `ios`,
`macos`, `linux`, `android`, and `any`. Set `any` to `true` only when the same
package supports every named host; it overrides false host-specific values. A
missing `platforms` object is supported for older packages and means Windows,
macOS, and Linux only. Hosts otherwise require their matching value to be
exactly `true`; they still validate the app's roles, entry points, capabilities,
and URLs. Claim each phone host separately unless `any` is accurate, because
working on Android does not prove iOS compatibility.

The current Android and iOS hosts support reviewed `SummerApp` pages,
`SuggestionProvider` modules, structured `WidgetProvider` renderers, bounded
app notifications, and manifest-defined ordinary or protected settings
registered at build time in
`packages/summer-android/src/summerApps/builtinMobileApps.ts`. Their runtime code
must be browser-safe TypeScript and must not import Node.js or Electron APIs.
Setting `platforms.android` or `platforms.ios` to `true`, or accurately claiming
`platforms.any`, declares eligibility for that host; it does not automatically
install the app. Mobile page apps are therefore compile-time bundled rather
than downloaded.

### Theme-aware suggestion artwork

Keep `suggestionImage` as the cross-version fallback used by older Summer
hosts. Apps may supplement it with light and dark variants; the suggestion grid
switches them immediately when the effective browser theme changes:

```json
{
  "suggestionImage": "data:image/svg+xml,...",
  "suggestionImageVariants": {
    "light": "data:image/svg+xml,...",
    "dark": "data:image/svg+xml,..."
  }
}
```

Either override may be omitted, in which case Summer uses `suggestionImage` for
that theme. These fields are image URLs consumed by browser chrome, not paths
resolved relative to `sap.json`; use an encoded `data:image/...` URL or an
absolute HTTPS URL. Use artwork with an intentional opaque or transparent
background; transparent pixels are composited over Summer's current theme
surface.

For the `SummerApp` role, `serve(request)` must return a `Response` with a
`text/html` content type and a body no larger than 1 MiB. The mobile shell
renders that
document in an iframe with a host-supplied Content Security Policy, scripts and
forms allowed, and same-origin access deliberately omitted. App page code cannot
reach Capacitor or the native website WebViews. Declarative settings and native
structured widgets are supported. Custom settings views, sandboxed widget HTML,
and downloaded third-party app code are not supported yet.

### Mobile bundling pipeline

Declaring a mobile platform makes an app eligible; it does not place the app in
either mobile build. Add reviewed apps explicitly to
`packages/summer-android/src/summerApps/builtinMobileApps.ts` with a manifest
import and a source-module loader. That registry is the mobile trust allowlist.

Vite follows each registered source import during `npm run build:android` or
`npm run build:ios` and emits dynamic imports as local `dist/assets/*` chunks.
`npm run android:sync` copies `dist/` into the Android project before Gradle
packages the APK; `npm run ios:sync` copies the same output into the Xcode
project before Xcode packages the app. Installed apps load those chunks locally;
they do not contact an app server or download package code.

The shared Vue shell selects the current Capacitor host and activates the
registry at browser startup. Mobile catalog admission and desktop package
installation use the same public SDK manifest validator before loading app code;
entry paths must remain inside the package. Mobile activation then loads the
module, calls `activate(context)`, and retains the active module. The current
startup eagerly activates every valid registry
entry even though Vite places dynamic imports in separate chunks. A
`SuggestionProvider` can then receive address-bar queries, while only an app
that declares `SummerApp` enters the `sap://` page catalog.

See `packages/summer-android/README.md` for the complete build and runtime flow.
Treat every new registry entry as trusted browser code and review its entire
dependency graph before bundling it.

The runtime entry must export `activate(context)` and `deactivate()`. An
openable page app can begin with:

```js
let appContext;

export function activate(context) {
  appContext = context;
  context.log("activated");
}

export function deactivate() {
  appContext = undefined;
}

export async function serve(request) {
  const {pathname} = new URL(request.url);
  if (pathname !== "/") {
    return new Response("Not found", {status: 404});
  }

  return new Response("<!doctype html><h1>Hello from Summer</h1>", {
    headers: {"content-type": "text/html; charset=utf-8"}
  });
}
```

Use one module format consistently. TypeScript source must compile to a
Node-loadable `.js` or `.mjs` entry; `sap.json` must point to emitted JavaScript,
not TypeScript source.

## Add capabilities deliberately

Use `offlineSupport.available` to declare that the app has useful functionality
without an internet connection. Set `offlineSupport.offerOnConnectionError` to
`true` only when the app also opts into Summer's offline error-page chooser.
This separation lets an app advertise offline support without appearing in that
recovery surface. Summer offers an app only when both values are `true`, the app
is enabled, and it declares the openable `SummerApp` role. These fields are not
permissions and do not imply that every app feature works offline.

For a reviewed legacy package that predates this manifest field, Summer's
bundled `public/appregistry.json` entry may carry the same `offlineSupport`
object as compatibility metadata. This fallback applies only when the installed
manifest omits `offlineSupport`; an explicit manifest declaration always wins.

Declare settings in `sap.json`, read them from `context.settings`, and dispose
change subscriptions during `deactivate()`. Never put a secret default in a
manifest. A declared secret automatically requires the host-neutral
`settings-secrets` capability; supported hosts deliver the value only to the
owning app runtime and show the settings UI only whether it is configured. See
[`summer-app-settings.md`](summer-app-settings.md).

Every app receives `context.storage`, a namespaced structured-data store, and
`context.crypto.randomBytes()`, a bounded OS CSPRNG capability. Stored values are
preserved when an app is disabled or updated and removed on authorized
uninstall.

OS credential storage is deliberately narrower. Only an exact canonical
bundled app whose manifest declares `permissions.access: ["credential-storage"]`
receives `context.credentials`. Credentials are app- and browser-instance-scoped,
returned as mutable byte copies, and removed only on authorized uninstall. A
third-party app cannot gain this capability by copying a bundled app ID or
`builtin` marker.

For user-visible status alerts, declare `capabilities: ["notifications"]` and
feature-detect `context.notifications`. `show()` accepts only an app-local ID,
title, and body and returns `shown`, `denied`, or `unavailable`. The host derives
the app identity itself. On phones, Summer also requires the user to enable
notifications for that app in the Apps Library and requires OS notification
permission; the native adapter revalidates both before delivery. Do not add URLs,
actions, origins, filenames, native options, or background registration to the
payload. Websites do not receive this capability.

To make app-owned operations discoverable by browser agents, declare
`capabilities: ["agent-tools"]`, feature-detect `context.agentTools`, and call
`context.agentTools.provide({id, tools}, invoke)` during `activate()`. Keep and
call the returned disposer during `deactivate()`; the desktop loader also
disposes registrations if activation fails or the app is unloaded. Descriptors,
schemas, inputs, results, provider size, global registry size, and execution
time are bounded by the browser-owned Tool Hub; one active app may register at
most 16 providers, and one invocation stops being awaited after ten seconds.
Apps receive no tab, DOM, Electron, Node, filesystem, or raw IPC authority from
this facet. An app controls its callback and cannot authorize itself with its
declared risk, so the trusted desktop boundary currently treats every
app-provided tool as `sensitive`. The user's AI Services agent policy may
allow, prompt, or block those tools; a native prompt identifies the app using
its browser-owned app ID and manifest name. The app cannot read, set, or opt out
of that decision. A timed-out callback has an unknown outcome and is never
retried automatically.

Declare every user-visible widget in the manifest. Native widgets return
supported structured blocks. Sandboxed widgets package an HTML entry and use
the restricted host bridge; allow only the exact HTTPS or WSS origins they
need. Validate widget IDs, action IDs, and payloads before privileged work.
See [`summer-app-widgets.md`](summer-app-widgets.md).

Suggestion results must be structured-clone-safe. Respect abort signals, bound
network work, and validate URLs or custom actions again at the privileged
boundary. See
[`summer-app-suggestions.md`](summer-app-suggestions.md).

An app may expose several named APIs through `manifest.apis` and
`context.provideAPI()`. Consumers use `context.loadAPI()` after declaring the
exact ID under `permissions.apis`; approved top-level HTTPS websites use
`window.summi.loadAPI()`. Every call is asynchronous, bounded, cancellable, and
JSON-shaped. See [`summer-app-apis.md`](summer-app-apis.md).

For a concrete provider/consumer pair, the bundled Mesh app declares
`summer-mesh-service` and registers it with `context.provideAPI()`. Mesh Lab
declares that exact ID under `permissions.apis` and calls `context.loadAPI()`.
This lets the lab request its caller-bound BAR address and validate user-supplied
peer identities without receiving Mesh private keys or raw transport access.

For `serve()`, normalize routes, return explicit status codes and content
types, and never reflect attacker-controlled content into HTML.

## Bundle an app with Summer

For a built-in app:

1. Put editable source, `sap.json`, and `package.json` under
   `packages/<app-id>/`.
2. Set `"builtin": true` in the manifest.
3. Give the package a `release` script that type-checks, builds into `dist/`,
   and runs the shared built-in app releaser. It refreshes only
   `public/builtin/sap/<app-id>/`.
4. Add `{"path":"/builtin/sap/<app-id>"}` to
   `public/builtin/appregistry.json`. A bundled experimental app may declare
   `"enabledByDefault": false`; this affects only its first installation, while
   package replacements preserve the user's current enabled state.
5. Add a focused `tests/<app-id>App.test.ts` that checks registry wiring,
   manifest identity and version alignment, entry existence, compilation, and
   generated-output freshness.

The root builder discovers immediate `packages/` children whose manifest has
`builtin: true` and whose package defines `scripts.release`. Do not add a
hard-coded root build alias for each app:

```sh
npm run build:sap -- <package-name-or-id>
npm run build:builtin-apps
```

Treat `public/builtin/sap/<app-id>/` as generated runtime output. Change the
package source and rebuild it; never fix only the public copy. Do not add an
ordinary app to `public/appregistry.json` until a real packaged release and
stable metadata exist. Never fabricate a download URL.

Use `scripts/release-builtin-app.mjs` instead of writing an app-specific copy
script. It releases `sap.json` and every regular file under the package's
`dist/` directory. If an app has static runtime assets that its compiler does
not emit, declare only those exceptions as data in `package.json`:

```json
{
  "summerApp": {
    "release": {
      "assets": [
        {"source": "src/widget/index.html", "release": "widget/index.html"}
      ]
    }
  }
}
```

The shared releaser validates that paths stay inside the package and its
generated app directory, preserves byte-identical files, and removes stale
outputs. `scripts/build-builtin-widget-app.mjs` provides the common TypeScript,
Vue type-check, Vite, watch, and release pipeline for sandboxed Vue widget apps.
Adding another app therefore requires package configuration, not another root
copy or build script.

A bundled desktop app may leave a bare runtime import such as
`@summer/app-sdk` in its generated server module. Every such package must be an
explicit production dependency of the root browser package, not only a
dependency of the app workspace. The packaged-runtime release gate follows the
relative module graph from every registered built-in app entry, checks each
bare import against the root production dependency graph, and then verifies
the generated app files, dependency manifests, and resolved package exports in
`app.asar`. This is a host guarantee for bundled desktop apps. Externally
installed app archives remain self-contained and must bundle their own runtime
dependencies.

An exact bundled system app may declare `management: {"uninstall": false}`.
Summer then permits enable and disable but rejects package replacement from an
untrusted path and rejects uninstall. Disabling always preserves runtime storage
and credentials. This field cannot make a third-party package non-removable.

## Protect lifecycle and trust boundaries

- Treat requests, manifests, settings, suggestion data, widget messages, URLs,
  paths, and remote responses as untrusted.
- Request the least privilege and validate privileged operations in the main
  process.
- Track every timer, listener, watcher, connection, and subscription created
  by `activate()` and release it in `deactivate()`.
- Make repeated activation and deactivation safe.
- Use only the bounded `AppContext` APIs unless a separately reviewed browser
  runtime change is required.
- Never expose Electron, Node.js, raw IPC, filesystem, process, credentials, or
  unrestricted networking to sandboxed content.
- Make custom UI keyboard-accessible, localizable, usable in narrow windows,
  and correct in light, dark, LTR, and RTL modes.

Summer CLI is a deliberately narrow browser-runtime exception, not a
general app capability. Its page-only preload and main-process service authorize
the exact bundled `sap://cli/` document on every operation; no other Summer
App can request or declare terminal access. See
[`summer-cli.md`](summer-cli.md) before changing that boundary.

## Verify the app

Run the narrowest applicable checks:

```sh
npm --prefix packages/<app-id> run build
npm run build:sap -- <app-id>
npm run test:unit -- tests/<app-id>App.test.ts
npm run eslint -- packages/<app-id> tests/<app-id>App.test.ts
npm run build:vue
```

Use `npm run build:builtin-apps` when the aggregate discovery and release path
also needs proof. For a plain example, at minimum parse `sap.json`, import the
runtime entry, and exercise activation, role-specific exports, and deactivation
in a focused test.

Confirm that every manifest path exists in packaged output. Exercise successful
and rejected routes or actions, lifecycle cleanup, settings defaults and
changes, and widget or suggestion normalization where relevant. Use the
Playwright harness or an explicit disposable `--instance-id` and
`--instance-data-dir` for manual testing; never use a real Summer profile.

Review store packaging and registry requirements in
[`summer-app-submission.md`](summer-app-submission.md) before publishing.
