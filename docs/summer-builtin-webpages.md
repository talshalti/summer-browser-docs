# Summer built-in webpages

Summer built-in webpages are browser-owned frontend pages exposed through
readable URLs such as `summer://welcome/`. They are packaged with the browser
and are separate from peer-to-peer content that also uses the `summer://`
protocol.

The planned documentation site's route, content, ownership, versioning, and
quality contract is recorded in
[`summer-docs-information-architecture.md`](summer-docs-information-architecture.md).

Summer currently ships these user-facing browser pages:

| URL | Purpose |
| --- | --- |
| `summer://downloads/` | Search and manage persisted downloads for the current profile |
| `summer://history/` | Opt into local history recording, then search, open, or delete saved visits |
| `summer://pdf/?source=…` | Render a validated local or web PDF in Summer's sandboxed browser-owned viewer |
| `summer://settings/` | Open a validated section in the browser-owned Settings panel |
| `summer://password/` | Redirect to the Passwords section of the browser-owned Settings panel |
| `summer://telemetry/` | Privacy controls and the planned usage-data catalog. New usage and AI-request collection is inactive in 2.10.0; the private-mode switch cuts communication with Summer-operated servers. |

These pages deliberately keep the generic internal bridge disabled. Downloads,
History, and PDF receive narrowly scoped preload APIs, while Settings only
requests a validated Settings destination from the owning regular browser window.
The PDF page receives only the source encoded in its own canonical URL; the main
process validates ownership, session, size, and the PDF header before returning
bytes to the sandboxed renderer. The renderer uses PDF.js's bounded page-view
buffer plus text and annotation layers, and ships the upstream CMap, standard
font, ICC, and WASM resources needed for document compatibility.
History recording is off by default and must be enabled by the user on the
History page. Turning recording off stops future capture without deleting
existing entries; deletion remains an explicit, separately confirmed action.
The browser-page copy currently has reviewed complete English and Hebrew packs.
Other interface locales deliberately use Vue-i18n's English fallback for these
new page keys; the locale validator permits only this declared fallback while
continuing to require pre-existing Downloads notification translations.

The shared data registry in
[`shared/summerBuiltinWebpages.json`](../shared/summerBuiltinWebpages.json) is
the single source of truth for these pages. The typed helpers in
[`electron/summerBuiltinWebpages.ts`](../electron/summerBuiltinWebpages.ts)
expose that data to Vite, Electron, and the preload code. A registered page
automatically becomes a Vite build entry, receives a root route, can load
built assets, and is recognized as browser-owned. Access to the privileged
internal preload bridge is a separate, explicit registry decision.

## How to add a new built-in webpage

Before creating files, inspect the registry, routing implementation, scaffold
script, registry tests, and a current sibling such as `ui/welcome.html` and
`ui/welcome.ts`. Choose a lowercase kebab-case hostname, define its visible
states and navigation entry points, and list any static resources or internal
browser capabilities it needs.

Confirm that the feature belongs in trusted browser-owned UI. Use the human
[`Summer App development guide`](summer-app-development.md) for an installable
app, and an ordinary webpage for untrusted or remote content.

### Quick scaffold (recommended)

Run the scaffold command with a lowercase kebab-case page ID:

```sh
npm run create:builtin-webpage -- release-notes
```

The command creates `ui/release-notes.html`, `ui/release-notes.ts`, and
`ui/release-notes.css`, then adds the page to the shared JSON registry. It
refuses to overwrite existing files or register an existing ID. The page is
immediately addressable as `summer://release-notes/` after building.

The steps below describe what the command creates and can also be followed
manually when a page needs a custom structure.

### 1. Create the frontend entry

Create an HTML entry point under `ui`. For a page named `release-notes`, the
minimal structure can be:

```text
ui/
|-- release-notes.html
|-- release-notes.ts
`-- release-notes.css
```

Load the TypeScript entry from the HTML using a relative module URL:

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Release notes</title>
</head>
<body>
    <main id="app"></main>
    <script type="module" src="./release-notes.ts"></script>
</body>
</html>
```

The TypeScript entry can import its stylesheet and initialize the page:

```ts
import "./release-notes.css";

document.querySelector("#app")!.textContent = "Release notes";
```

Preserve a restrictive Content Security Policy. Do not add inline scripts,
`unsafe-eval`, remote scripts, or broad network origins. Use semantic
landmarks, labels, real buttons and links, visible focus states, logical tab
order, and appropriate live regions for asynchronous status. Support text zoom,
narrow windows, keyboard-only use, light and dark themes, and LTR and RTL
direction.

Put user-visible text behind the existing i18n system and update every
applicable locale. Set document `lang` and `dir`; follow `ui/welcome.ts` when
the page must react to live locale changes. Keep feature logic testable outside
DOM handlers and clean up timers, observers, subscriptions, and global
listeners.

### 2. Register the page

Add the HTML entry to `shared/summerBuiltinWebpages.json`:

```json
{
  "welcome": {
    "entryPath": "/ui/welcome.html",
    "internalBridge": true
  },
  "release-notes": {
    "entryPath": "/ui/release-notes.html",
    "internalBridge": false
  }
}
```

The new page is available at:

```text
summer://release-notes/
```

Registry IDs must be lowercase URL hostnames. Prefer kebab-case for IDs with
multiple words. `entryPath` must start with `/` and point to the HTML file as
it appears in the Vite distribution directory. The scaffolder sets
`internalBridge` to `false`; keep that secure default unless the page has a
reviewed need for an existing internal browser capability.

No separate Vite input or protocol handler is required. `vite.config.ts`
creates build inputs from the registry, and the `summer` protocol handler
resolves registered hosts through the same definitions.

### 3. Add optional page-specific files

Nested URLs use a directory with the same name as the HTML entry. For the
`/ui/release-notes.html` entry, this URL:

```text
summer://release-notes/changelog.json
```

first looks for:

```text
dist/ui/release-notes/changelog.json
```

Static files that need to be copied unchanged can be placed under
`public/ui/release-notes/` in the source tree. Vite copies `public` into
`dist`, producing the expected runtime directory. Files imported by the
TypeScript entry are handled by Vite and normally emitted under `dist/assets`.

### 4. Open the page

Use the registry helper when opening the page from TypeScript:

```ts
const releaseNotesUrl = getSummerBuiltinWebpageUrl("release-notes");
```

The resulting URL is `summer://release-notes/`. It can also be entered
directly in the browser's address bar.

### 5. Build and verify

Run `npm run build:vue`, confirm that `dist/ui/release-notes.html` exists, and
open `summer://release-notes/` in Summer. See
[Building and testing](#building-and-testing) for the relevant test command.

## Request routing

At runtime, `resolveSummerBuiltinWebpagePath` maps each hostname to its entry
and a sibling resource directory:

| Request URL | Behavior |
| --- | --- |
| `summer://welcome/` | Registry resolves `/ui/welcome.html` |
| `summer://release-notes/` | Registry resolves `/ui/release-notes.html` |
| `summer://welcome/whatever.page` | Registry first checks `/ui/welcome/whatever.page` |
| `summer://release-notes/images/header.png` | Registry first checks `/ui/release-notes/images/header.png` |
| `summer://password/` | Redirects to `summer://settings/#/passwords` |

The resource directory is derived by removing the extension from `entryPath`.
For example, `/ui/welcome.html` owns `/ui/welcome/`. When a scoped resource
does not exist, the protocol handler falls back to the original request path.
This lets generated references such as `/assets/page.js` continue to resolve
from `dist/assets` while page-specific files remain naturally grouped beneath
their page's directory.

An unregistered hostname does not match a built-in page. Its request continues
through the existing `summer://` fallback, preserving the protocol's
peer-to-peer behavior.

## Trust and preload access

Registering a page is also a trust decision. Registered URLs are recognized by
the preload code as browser-owned pages. They receive the internal Summer
bridge used by packaged browser UI only when their registry entry explicitly
sets `internalBridge` to `true`.

Only register HTML and scripts maintained and shipped by the browser. Do not
register remote, downloaded, peer-provided, or otherwise untrusted content.
If a page does not need internal browser APIs, consider whether it should be a
normal webpage instead of a built-in one, and always leave `internalBridge`
disabled. Enabling the bridge exposes `$summer` and generic `$electron` IPC, so
it requires review of every renderer action and corresponding main-process
authorization.

Treat URL parameters, imports, remote responses, messages, and form values as
untrusted even on a browser-owned page. Prefer an existing bounded internal API.
When a new capability crosses a process boundary, update the main handler,
preload bridge, shared or global type, renderer caller, and contract tests
together. Validate sender and origin and authorize the operation in
`electron/main/`; renderer validation is not sufficient.

Never expose raw IPC, Electron, Node.js, filesystem, shell, credentials, or
unrestricted networking to page code. Do not render remote or peer-provided
HTML as trusted markup. Avoid `innerHTML`; if rich content is unavoidable, use
a reviewed sanitizer and a tightly bounded input format.

## Registry helpers

The registry module exports these helpers:

- `getSummerBuiltinWebpageUrl(id)` creates a page's public root URL.
- `matchSummerBuiltinWebpage(url)` returns its registry definition and parsed
  URL, or `null` when it is not registered.
- `isSummerBuiltinWebpage(url)` checks whether a URL belongs to a registered
  built-in page.
- `resolveSummerBuiltinWebpagePath(url)` maps a registered URL to its file path
  inside `dist`.

Use `getSummerBuiltinWebpageUrl` instead of repeating URL strings in backend
code:

```ts
const releaseNotesUrl = getSummerBuiltinWebpageUrl("release-notes");
```

The ID argument is inferred from the registry keys, so TypeScript rejects IDs
that have not been registered.

## Building and testing

Type-check and build the frontend after adding a page:

```sh
npm run build:vue
```

The generated HTML should appear under `dist/ui`. Run the registry unit tests
with:

```sh
npx vitest run tests/summerBuiltinWebpages.test.ts
```

When adding routing behavior, extend
[`tests/summerBuiltinWebpages.test.ts`](../tests/summerBuiltinWebpages.test.ts)
with both matching and non-matching URL cases.

Add a focused unit test for pure page logic. Use Playwright when navigation,
internal actions, focus order, or layout behavior cannot be proved by a unit
test. Exercise every applicable loading, empty, success, failure, and retry
state, plus keyboard use, narrow layout, themes, and RTL.

For a manual browser check, use the Playwright harness or explicit disposable
`--instance-id` and `--instance-data-dir` arguments. Never test against a real
Summer profile.

Before finishing, confirm that the registry remains the single source of truth,
every referenced asset exists in the build, privileged actions are bounded and
authorized, translations and accessibility states are complete, and no
generated output or unrelated file entered the change.
