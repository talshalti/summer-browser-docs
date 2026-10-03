# Summer App settings

> Implementation status: the declarative form, validation, persistence, runtime
> change API, sandboxed desktop custom views, and bounded desktop Settings
> sidebar entries are implemented. Packaged i18n resources and mobile custom
> views remain planned.

## Quick start

Add a small declaration to `sap.json`:

```json
{
  "name": "My App",
  "id": "my-app",
  "main": "./index.mjs",
  "version": "1.0.0",
  "type": ["SummerApp"],
  "settings": {
    "schemaVersion": 1,
    "fields": {
      "notificationsEnabled": {
        "type": "boolean",
        "label": "Enable notifications",
        "default": true
      },
      "refreshMinutes": {
        "type": "number",
        "label": "Refresh every (minutes)",
        "default": 15,
        "min": 1
      }
    },
    "sections": [
      {
        "id": "general",
        "title": "General",
        "fields": ["notificationsEnabled", "refreshMinutes"]
      }
    ]
  }
}
```

Validate the declaration with the same policy used by desktop and mobile:

```sh
npx summer-app-conformance sap.json --host android
npx summer-app-conformance sap.json --host ios
```

Host-neutral tools may also import `normalizeAppSettings`,
`validateAppSettingValue`, and `validateAppSettingsPatch` from
`@summer/app-sdk/settings`.

Read the values once, then listen for changes:

```ts
let unsubscribe: (() => void) | undefined;

export function activate(context: AppContext) {
  const apply = (settings: Readonly<Record<string, unknown>>) => {
    // Apply settings to your app here.
  };

  apply(context.settings.getAll());
  unsubscribe = context.settings.onDidChange(({values}) => apply(values));
}

export function deactivate() {
  unsubscribe?.();
}
```

Summer Apps may provide a native form generated from `sap.json` and sandboxed
desktop custom views. Packaged localization resources remain planned; localized
text declarations use a known browser-interface translation when their key
exists in the selected language, otherwise their fallback text.

Any installed app role may declare settings. Summer owns validation,
persistence, accessibility, theming, and change notifications.

## Manifest

```json
{
  "name": "Example Service",
  "id": "example-service",
  "main": "./index.mjs",
  "version": "1.0.0",
  "type": ["SummerApp", "SuggestionProvider"],
  "settings": {
    "schemaVersion": 1,
    "layout": {
      "contentWidth": "wide",
      "columns": "auto",
      "density": "comfortable",
      "labelPosition": "top"
    },
    "fields": {
      "provider": {
        "type": "select",
        "label": {"key": "provider.label", "fallback": "Provider"},
        "default": "summer",
        "options": [
          {"label": "Summer", "value": "summer"},
          {"label": "Custom", "value": "custom"}
        ]
      },
      "apiUrl": {
        "type": "string",
        "label": {"key": "apiUrl.label", "fallback": "API URL"},
        "visibleWhen": {
          "field": "provider",
          "operator": "equals",
          "value": "custom"
        }
      },
      "refreshMinutes": {
        "type": "number",
        "label": {"key": "refresh.label", "fallback": "Refresh interval"},
        "default": 15,
        "min": 1,
        "max": 1440
      },
      "apiToken": {
        "type": "secret",
        "label": {"key": "token.label", "fallback": "API token"}
      }
    },
    "sections": [
      {
        "id": "connection",
        "title": {"key": "connection.title", "fallback": "Connection"},
        "layout": {
          "columns": 2,
          "rows": [
            [{"field": "provider"}, {"field": "refreshMinutes"}],
            [{"field": "apiUrl", "span": 2}],
            [{"field": "apiToken", "span": 2}]
          ]
        }
      }
    ]
  }
}
```

The existing flat `settings` field map may be accepted as shorthand for a
version 1 declaration with those entries under `fields`.

## Native form

Supported field types are `boolean`, `string`, `number`, `select`, and
`secret`. Common properties are `label`, `description`, `default`, `required`,
and `visibleWhen`. Type-specific validation includes string lengths/patterns,
number ranges/steps, and fixed select options.
String fields may set `multiline: true` for lists, templates, or other longer
configuration values.

Layout is constrained so forms remain responsive and accessible:

- Form hints: `contentWidth`, `columns`, `density`, and `labelPosition`.
- A section may list fields in order or define explicit `layout.rows`.
- A section with `advanced: true` is collapsed behind an advanced-settings disclosure.
- A row cell names a field and may span multiple columns.
- Each field must appear exactly once; duplicates and unknown IDs are errors.
- Narrow windows flatten rows in reading order. Summer may override layout for
  RTL, text zoom, touch input, or accessibility.
- Apps cannot provide arbitrary CSS, pixels, colors, or grid coordinates.

Native strings may be literal or `{key, fallback}` references. The current
renderer uses `fallback`; packaged locale resources are normalized but not
loaded. Authors should therefore write a complete fallback string for every
localized label.

Declarative definitions are static. `visibleWhen` supports simple dynamic
visibility based on other settings, and values update live. Runtime-generated
fields and options are not supported by the current native form.

## Custom desktop view

An app can contribute one app-relative page before, after, or instead of its
native form:

```json
"customView": {
  "path": "/settings",
  "label": "Connection status",
  "placement": "after-form"
}
```

Summer loads only the exact declared path through the browser-owned
`summer-app-settings:` proxy. The response is size-bounded, must be HTML, gets a
host CSP that denies network, forms, objects, and nested frames, and runs in an
opaque iframe sandbox without same-origin access. Inline scripts may use the
narrow `postMessage` bridge. Send
`{type:"summer-app-settings:ready",version:1}` to receive a non-secret state
snapshot. Updates use
`{type:"summer-app-settings:update",version:1,requestId,patch}` and resets use
`{type:"summer-app-settings:reset",version:1,requestId,keys}`. Summer validates
the source frame, request ID, concurrency, declared fields, and values before
persisting anything. Secret values are never sent into the frame.

The desktop proxy also installs a browser-owned size observer in each custom
view. It reports document-height changes to browser chrome so ordinary settings
pages grow with their content instead of adding a nested scrollbar. Browser
chrome accepts reports only from the exact current frame, ignores non-finite
values, and clamps the resulting height to 270–1,600 CSS pixels. Larger pages
remain scrollable inside that bounded frame.

Custom views are unavailable on mobile hosts for now; keep essential settings
available through native manifest fields.

## Main Settings sidebar entry

An enabled Summer App may expose its settings panel as a first-class entry in
Summer's desktop Settings sidebar. Add a bounded `sidebar` declaration to the
structured settings manifest:

```json
{
  "settings": {
    "schemaVersion": 1,
    "sidebar": {
      "label": "Network",
      "placement": "before-apps"
    },
    "fields": {
      "endpoint": {
        "type": "string",
        "label": "Endpoint"
      }
    }
  }
}
```

`placement` is either `before-apps` or `after-apps`, relative to the
browser-owned **Apps & Widgets** entry. Summer displays at most eight enabled
app-owned entries, prioritizes trusted built-ins when that bound is reached,
and removes an entry immediately when its app is disabled or uninstalled. The
entry opens the same validated native form and sandboxed custom content used by
the app's App info settings tab; it does not grant a new privileged API.
Browser-owned hubs may group closely related entries from the same trusted
built-in app without changing the app settings contract or its security
boundary.

## Additional desktop Settings pages

An app may declare up to four `settings.pages`, independently of its existing
native form, `customView`, and `sidebar`. Each page normally creates an entry
within the same eight-entry global sidebar bound. Mesh is the browser-owned
exception: its **Mesh network** and **My Devices** surfaces appear as tabs in
one **Link** Settings entry:

```json
"pages": [{
  "id": "my-devices",
  "label": {"key": "myDevices.title", "fallback": "My Devices"},
  "placement": "before-apps",
  "path": "/my-devices",
  "actions": {"invite": "/actions/my-devices/invite"},
  "navigation": "installed-apps"
}]
```

IDs and page paths must be unique. Page and action paths are bounded, literal
app-relative paths without query strings, fragments, encoded characters, or
traversal segments. Each page may declare at most sixteen named POST actions.
The desktop loads `summer-app-settings://<app-id>/<path>` into the existing
opaque `sandbox="allow-scripts"` frame. It retains the same restrictive CSP;
the frame cannot fetch, submit forms, navigate other frames, or call Electron.
App requests from this bridge include `x-summer-settings-view: 1`, allowing an
app to render a bridge adapter instead of its normal standalone page.
Mesh uses this marker to omit the standalone My Devices link from its Identity
view, because iframe navigation to `sap:` is blocked. Use the browser-owned
**Link → My Devices** tab instead.
The host also supplies `accept-language` from its selected interface locale on
initial GET, POST, reload, and same-page redirect requests. It does not forward
an app-supplied language header. A language change recreates the frame, and
late replies to the previous frame are discarded. The **My Devices** title has
English, Hebrew, and Arabic copy; other desktop packs currently keep its English
fallback label until reviewed translations are available.

After the existing `summer-app-settings:ready` handshake, the frame may send
these version-1 messages to its parent with a unique `requestId` (1–64 ASCII
letters, digits, underscores, or hyphens):

- `summer-app-settings:action`, with a declared `action` ID and `fields`, a
  record of string values. The host accepts at most 32 fields, 8,192 characters
  per value, and 16 KiB after URL encoding.
- `summer-app-settings:reload`, with no action or fields. The host GETs only the
  same declared page. Apps can replace a status region without resetting input.
- `summer-app-settings:open-app`, with an installed `appId` and optional
  `fragment` (at most 256 characters, no controls or spaces). This requires
  `navigation: "installed-apps"` and active user interaction in browser chrome.
  The host validates that the target is enabled and constructs only its fixed
  `sap://<app-id>/` root plus the fragment, opening it in a normal tab.

Action and reload responses use `summer-app-settings:action-result` with
`version: 1`, `requestId`, `ok: true`, `html`, and `status`. Navigation success
has no HTML. Failures have `ok: false` and `error`. The app owns rendering this
HTML within its existing sandbox; browser chrome never inserts it into its DOM.
HTML is UTF-8 and bounded to 512 KiB. A POST may redirect once with HTTP 303
back to the exact declared page; all other redirects are rejected. The main
process independently validates declarations, enabled state, input sizes, and
a maximum of four concurrent requests per chrome window. Private windows cannot
invoke this bridge, so additional app pages are omitted from their sidebar.
For Mesh, **Link** therefore retains only the **Mesh network** tab in a private
window. Existing native app settings entries remain available. A directly
selected additional page shows the existing private-window restriction message
instead of creating an iframe, and any stale frame request receives an immediate error.
Closing or changing pages discards late renderer replies.

Use this page bridge for transient state and user actions. Keep persistent
configuration in the existing native settings API. Apps must continue to
authorize their own actions and bind forms to their current app/profile state;
declaring a route does not replace those checks. Mobile hosts do not render
these desktop pages.

## App access and change notifications

Summer adds a settings API scoped to the app's `AppContext`:

```ts
interface AppSettingsApi {
  get<T = unknown>(key: string): T | undefined;
  getAll(): Readonly<Record<string, unknown>>;
  update(patch: Record<string, unknown>): Promise<void>;
  reset(keys?: string[]): Promise<void>;
  onDidChange(listener: (change: SettingsChange) => void): () => void;
}

interface SettingsChange {
  changedKeys: string[];
  values: Readonly<Record<string, unknown>>;
  source: "native-form" | "custom-view" | "app-runtime" | "reset";
}
```

```ts
let stopListening: (() => void) | undefined;
let settingsApi: AppSettingsApi | undefined;

export function activate(context: AppContext) {
  settingsApi = context.settings;
  applySettings(context.settings.getAll());

  stopListening = context.settings.onDidChange(change => {
    applySettings(change.values, change.changedKeys);
  });
}

export async function useCustomProvider() {
  if (!settingsApi) throw new Error("The app is not active");
  await settingsApi.update({provider: "custom"});
}

export function deactivate() {
  stopListening?.();
  stopListening = undefined;
  settingsApi = undefined;
}
```

`update()` persists and validates the change, then notifies the app runtime
and refreshes the native settings surface. There is no separate notification
method.

| Origin | Runtime event source |
| --- | --- |
| Native form | `native-form` |
| Custom view | `custom-view` |
| App runtime | `app-runtime` |
| Reset | `reset` |

Settings are for persisted configuration, not transient status such as
progress, connection health, or unread counts.

## Profile language preferences

Summer Apps also receive a browser-owned, read-only language capability:

```ts
interface ProfileLanguagePreferencesApi {
  get(): {
    readonly understoodLanguages: readonly string[];
    readonly translationTarget: string;
    readonly offerTranslation: boolean;
  };
  onDidChange(listener: (preferences: ProfileLanguagePreferences) => void): () => void;
}
```

Use `context.languagePreferences` when an app should follow the browser
profile's language choices. An app may expose a separate custom mode, but must
not mutate the profile through this API. Canonicalization, language matching,
Captain Word integration, and page-translation behavior are documented in
[`language-preferences-and-translation.md`](language-preferences-and-translation.md).

## Storage and safety

- Desktop stores values per Summer profile and app ID. Mobile stores them per
  app installation and app ID; downloaded app installation/uninstall remains
  outside the current mobile allowlist.
- Updates and package replacements preserve settings. Desktop uninstall removes
  them.
- Defaults are read from the current manifest and are not stored as overrides.
- Ordinary desktop overrides use the existing atomic profile store. Ordinary
  mobile overrides use bounded trusted app storage with validated read-back;
  secret fields are never serialized there.
- Desktop secret fields use its existing OS credential store. Android encrypts
  a separate app-ID/field-ID record with a non-exportable Keystore AES-GCM key
  and address-bound additional authenticated data. iOS uses a separate generic
  password Keychain service with `WhenUnlockedThisDeviceOnly` and synchronization
  disabled.
- A multi-key update spanning ordinary and protected stores is not one
  transaction and can partially persist if a later write fails.
- The SDK infers `settings-secrets` whenever a manifest declares a secret field.
  A host must advertise that capability or reject the app before loading code.
- The scoped `AppContext.settings` API exposes only the calling app's declared
  keys. The owning runtime receives its secret values, while host-facing
  snapshots contain ordinary values plus configured secret field IDs only.
- Secret values never enter the host settings snapshot, ordinary renderer
  storage, exports, or logs. Capacitor method-payload logging is disabled on
  mobile.
- Labels and descriptions are rendered as text, never app-provided HTML.

## Remaining planned work

- Load and validate packaged settings locale resources instead of always using
  localized-text fallbacks.
- Add the equivalent sandboxed custom-view host to mobile.
