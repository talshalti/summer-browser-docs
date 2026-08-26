# Desktop-to-mobile capability reference

Status: working engineering guide for the [desktop-to-mobile strategy](desktop-to-mobile-strategy.md)

> **Version 57 WebExtension status:** generic signed-package management remains
> implemented, but downloaded code executes only after native proves engine
> ownership, isolated-world CSP, and package-format support. Android WebView and
> iOS 15–18.3 fail closed with `unsupported-engine`; iOS 18.4+ native
> `WKWebExtension` integration is in progress. Android execution is blocked by
> the current no-emulation/no-alternate-engine constraint.
> All older mobile WebExtension capability rows are historical unless an
> engine-backed device gate proves them. See
> [the engine migration plan](mobile-web-extension-engine-migration.md).
Last implementation audit: 2026-08-13
Audience: Tal, Codex, app authors, and future maintainers

## Purpose

This document is the detailed working guide behind our mobile blueprint. Use it
to choose a capability, locate the reusable policy, define the native boundary,
and decide what evidence proves Android/iOS parity.

The primary strategy records direction and scope. This reference records the
engineering detail and should change as implementation knowledge improves. It
is not a promise that every listed capability belongs in the next release.

## Contents

- [Reuse classes](#reuse-classes)
- [Repository reality](#repository-reality)
- [Target packages and dependency rules](#target-package-and-dependency-rules)
- [Feature-transfer playbook](#feature-transfer-playbook)
- [Capability matrix](#capability-matrix)
- [Capability-specific notes](#capability-specific-notes)
- [Data ownership and current scope](#data-ownership-and-current-scope)
- [Summer Apps transfer](#summer-apps-transfer)
- [File-level change map](#file-level-change-map)
- [Parity evidence](#parity-evidence-for-each-slice)
- [Risks and controls](#risks-and-controls)
- [Verification command map](#verification-command-map)

## Reuse classes

| Class | Meaning | Typical work |
| --- | --- | --- |
| A - direct | Already pure or portable TypeScript | Move or import with focused tests |
| B - extract | Useful behavior is mixed with Vue, IPC, Electron, or storage | Capture behavior in fixtures, then extract models, validation, reducers, and policy |
| C - native adapter | Requires a browser engine or operating-system service | Define a narrow TypeScript contract and implement Electron, Android, and iOS adapters |
| D - redesign or omit | Desktop interaction or runtime does not map safely to mobile | Design a mobile workflow, defer it, or keep it desktop-only |

A capability can use more than one class. Downloads, for example, use shared
TypeScript state and policy (B), native execution (C), and phone-specific
presentation (D).

## Repository reality

The target package design must start from the code that exists now:

| Current source | What it tells us |
| --- | --- |
| [`package.json`](../package.json) | Workspaces are explicit; the foundation adds `packages/browser-core` and `packages/summer-app-sdk` plus focused quality scripts |
| [`types/summerApps.d.ts`](../types/summerApps.d.ts) | Internal compatibility barrel now re-exports pure SDK type modules and retains only Electron installation records and IPC envelopes; public contracts contain no `WebContents` IDs |
| [`shared/summerAppPlatforms.ts`](../shared/summerAppPlatforms.ts) | Compatibility re-export preserves existing desktop/mobile imports while the portable implementation lives in `summer-app-sdk` |
| [`mobileSummerAppRuntime.ts`](../packages/summer-android/src/summerApps/mobileSummerAppRuntime.ts) | The public SDK owns shared manifest, module, context, lifecycle, and compatibility contracts; mobile retains only bounded host projections, results, and native adapters |
| [`sap.schema.json`](../sap.schema.json) | The checked-in manifest schema is the current external contract and must not drift from exported SDK types/validators |
| [`packages/mesh-identity`](../packages/mesh-identity) | Existing pattern for a compiled TypeScript workspace with explicit package exports and focused package tests |
| [`packages/widget-vue`](../packages/widget-vue) | Existing internal source-export package pattern for Vue-specific reuse |

The public SDK is the portable intersection. App authors import it directly;
`types/summerApps.d.ts` remains an internal compatibility barrel containing only
Electron installation records and IPC envelopes alongside SDK type re-exports.

## Target package and dependency rules

Foundation paths and target growth:

```text
packages/
|-- browser-core/          # private workspace package
|   |-- src/navigation/
|   |-- src/suggestions/
|   |-- src/tabs/
|   |-- src/repositories/
|   `-- src/hosts/
`-- summer-app-sdk/        # independently versioned, publishable package
    |-- src/manifest/
    |-- src/lifecycle/
    |-- src/settings/
    |-- src/suggestions/
    |-- src/pages/
    |-- src/widgets/
    |-- src/testing/
    `-- src/compatibility/
```

The paths are proposals; the dependency boundaries are requirements:

| Consumer | May depend on | Must not depend on |
| --- | --- | --- |
| External Summer App | Public Summer App SDK | Browser core, Electron, Capacitor, private stores, signing/release code |
| Public Summer App SDK | Platform-neutral libraries and Web-standard types | Browser core, Vue, Electron, Capacitor, Node-only APIs, Java, Swift |
| Browser core | Public SDK contracts where app behavior intersects the browser | Electron, Capacitor, Java, Swift, renderer singleton stores |
| Desktop renderer/main adapters | Browser core and public SDK | Mobile native implementations |
| Mobile Vue/Capacitor adapters | Browser core and public SDK | Electron and desktop-only contexts |
| Android/iOS native hosts | Serialized capability contracts | Product policy and generic app execution |

Start `browser-core` as private. Give it focused build, typecheck, test, and
explicit-export commands. Publish only the Summer App SDK because it has a real
external consumer. The final npm scope/name remains an explicit decision.

The SDK should own or generate the canonical portable manifest schema. Keep a
root compatibility path if existing tools require `sap.schema.json`, but do not
manually maintain two schemas.

## Feature-transfer playbook

### Step-by-step workflow

For each desktop capability:

1. Write the user outcome and current desktop behavior.
2. Locate policy mixed with Vue stores, Electron, IPC, persistence, or native
   callbacks.
3. Capture current behavior in deterministic fixtures before moving it.
4. Extract pure models, validation, reducers, and policy into `browser-core`.
5. Define the smallest host and repository ports the policy needs.
6. Wrap the existing desktop implementation without changing its behavior.
7. Implement matching Android and iOS adapters and phone-appropriate Vue UI.
8. Test success, cancellation, denial, failure, stale events, restart, recovery,
   and bridge isolation where applicable.
9. Update the capability matrix and record the commands that prove parity.

This is gradual replacement. No flag day is required.

### Implementation card

Copy this card into the task or pull-request description for every slice:

```text
Feature:
User outcome:
Current desktop owners:
Current mobile owners:
Behavior fixtures to preserve:
Shared TypeScript extraction:
Host ports and events:
Repository/migration impact:
Phone/tablet UX states:
Security boundary and attacker-controlled inputs:
Android proof:
iOS proof:
Desktop regression proof:
Explicitly out of scope:
Documentation updated:
```

A card should describe one reviewable vertical slice. If its host-port section
contains unrelated capabilities, split it.

### Pattern A: share pure behavior

Before this foundation, desktop and mobile implemented the same suggestion policy
separately. The foundation moves the real Fuse configuration, exact quick-key
matching, group ordering, and app-visibility policy into `browser-core`, with
fixtures that lock the shared answers and a desktop adapter regression test.

The implemented boundary is:

```ts
export type AddressSuggestionKind = "tab" | "bookmark" | "app" | "provided";

export interface AddressSuggestionDocument<T> {
  id: string;
  kind: AddressSuggestionKind;
  title: string;
  value: T;
}

export function filterAddressSuggestions<T>(
  suggestions: readonly AddressSuggestionDocument<T>[],
  query: string,
): AddressSuggestionDocument<T>[];

export function findExactAddressSuggestionQuickkey<T>(
  suggestions: readonly AddressSuggestionDocument<T>[],
  query: string,
): AddressSuggestionDocument<T> | null;
```

Desktop adapters obtain tabs, bookmarks, and provider results from Electron
services. Mobile adapters obtain them from mobile repositories and the bundled
app runtime. Both call the same policy and render the result differently.
Provider fetching, query-scoped refresh, and host-specific list limits remain in
host composition until their contracts are extracted.

Implemented movement:

```text
ui/store/suggestions.ts --------------------\
                                              -> packages/browser-core/src/addressSuggestions.ts
packages/summer-android/src/domain/          /
  addressSuggestions.ts --------------------/
```

Leave source adapters and Vue state composition in their hosts. Move only the
behavior that produces the same answer from the same input.

### Pattern B: share policy around native behavior

A web tab must use different native engines, but its logical state and event
rules can still be shared:

```ts
export interface TabSnapshot {
  tabId: string;
  revision: number;
  url: string | null;
  title: string | null;
  loadState: "idle" | "loading" | "failed" | "crashed";
  progress: number;
  canGoBack: boolean;
  canGoForward: boolean;
  active: boolean;
}

export type TabEvent =
  | { type: "upsert"; tab: TabSnapshot }
  | { type: "closed"; tabId: string; revision: number };

export interface TabHost {
  list(): Promise<readonly TabSnapshot[]>;
  create(url: string): Promise<TabSnapshot>;
  navigate(tabId: string, url: string): Promise<void>;
  goBack(tabId: string): Promise<void>;
  goForward(tabId: string): Promise<void>;
  close(tabId: string): Promise<void>;
  subscribe(listener: (event: TabEvent) => void): () => void;
}
```

One reducer accepts generation-and-sequence envelopes around `TabEvent`.
Electron now observes `WebContents` through a non-invasive adapter; the
Capacitor adapter maps `SummerTabs`; Java owns Android `WebView`; Swift owns iOS
`WKWebView`. Android and iOS stamp every command result and native callback with
one process generation and a monotonic sequence. The desktop adapter generates
the same envelope semantics without replacing desktop tab ownership. Revisions
reject stale state within a generation; generation IDs prevent callbacks from a
destroyed/recreated host from being accepted by the current host.

### Anti-patterns

- Importing Electron or Capacitor into shared policy.
- Publishing desktop `AppContext` types that contain `WebContents` IDs.
- Copying a desktop store and changing only its persistence calls.
- Giving website content a generic native or Capacitor bridge.
- Creating independent Android and iOS product policy.
- Declaring mobile compatibility based only on `platforms` without checking
  roles and required host capabilities.
- Adding a second mobile-only manifest for information already declared.
- Combining several privileged capabilities into one generic native command.

## Capability matrix

| Capability | Current mobile position | Target transfer | Class | Product cut |
| --- | --- | --- | --- | --- |
| Address input and URL safety | One `browser-core` resolver now drives mobile and both desktop address submission paths, canonicalizing HTTP(S), upgrading validated host-like input, selecting portable search URLs, and rejecting credentials, malformed hosts, and unsupported schemes | Keep host-specific search-provider execution outside the resolver; add any future navigable scheme only through an explicit validated host adapter | A/B | Foundation |
| Tabs and navigation | Native vertical slice, native Android/iOS generation/sequence envelopes, runtime validation, shared stale-event reducer, and an observational Electron adapter over the same snapshot contract implemented | Retain packaged lifecycle and desktop adapter evidence as adapters evolve | B/C | Daily MVP |
| Tab restoration and kept tabs | Versioned logical session, shared verified primary/recovery persistence with bounded corrupt-byte quarantine, host-configurable bounds, independent per-tab failure isolation, kept-tab state, Android/iOS termination/relaunch evidence, and a desktop logical-session adapter that leaves Electron navigation-stack persistence intact implemented | Keep packaged lifecycle and lossless desktop-adapter evidence current as adapters evolve | B/C | Daily MVP |
| Address suggestions | Shared `browser-core` document projection and matcher plus shared SDK provider timeout, cancellation, validation, ordering, and refresh orchestration consumed by desktop and mobile | Keep host presentation, destination validation, and trusted-only desktop exclusivity behind adapters as providers evolve | B | Foundation |
| Bookmarks | Shared versioned repository, verified dual-copy writes and corruption recovery, bounded corrupt-byte quarantine, verified legacy migration, CRUD/suggestions, explicit HTML import/export, cross-host selection and duplicate/failure accounting, configurable host bounds, and a lossless desktop adapter that preserves metadata/artwork while projecting portable fields implemented | Keep host-specific persistence and desktop-only fields behind their adapters as the portable schema evolves | B/C | Daily MVP |
| History | Intentionally excluded from mobile | Do not add a History tab, persist visits, or feed history suggestions; session URLs exist only to restore open tabs | D | Excluded |
| Search engines | Persistent Google, Bing, and DuckDuckGo mobile selection plus shared bounded query URL generation consumed by desktop and mobile implemented | Keep engine selection host-owned until desktop product settings intentionally add it; extend the shared generator for any new common engine | B | Daily MVP |
| Themes and localization | Persistent system/light/dark theme and English/Hebrew/Arabic selection implemented; one shared packaged flow changes theme, RTL locale, and search engine, then proves localized current values survive a full app stop/relaunch on Android and iOS simulators | Continue sharing tokens and message schemas while retaining form-factor-specific layouts | A/B | Daily MVP |
| Settings | Trusted persistent mobile surface implemented for daily-browser settings, notification opt-in, and confirmed cookie/cache clearing; the same verified dual-copy policy used by sessions, bookmarks, and downloads repairs missing/corrupt settings from the validated recovery slot; theme, language, and search controls expose stable localized current-value labels, and matching Android/iOS packaged evidence proves dark/Hebrew/DuckDuckGo persistence plus no History across relaunch; the responsive Library keeps the stacked phone navigation below tablet dimensions and uses a logical-direction two-column split on tablets, with matching packaged Android tablet-dimension and iPad mini evidence across every Library section, Settings, close-to-browser, and no History; desktop now projects only semantically portable theme/locale/search values while preserving host-only state | Grow the validated schema per capability without exposing generic native storage or visit storage | B/C | Daily MVP-Public preview |
| Built-in Summer pages | The host-neutral TypeScript Summer App Guide is discoverable in the Apps Library and renders in a constrained trusted-app workflow on Android/iOS | Add eligible responsive page apps without moving website content into the Vue bridge surface | B/C | Public preview |
| Downloads | Shared `browser-core` snapshot validation, strict declared-field projection, recovery-candidate derivation, verified dual-copy durable repository, bounded corrupt-byte quarantine, and process-loss reconciliation drive the mobile TypeScript and Capacitor boundaries; bounded Android/iOS adapters, trusted Downloads UI, native completed-file system presentation by opaque ID, packaged Android/iOS presentation/cancellation/failure/restart evidence, Android `DownloadManager` job reattachment, and iOS background `URLSession` recovery for eligible HTTPS/local-network GET transfers are implemented | Keep both simulator flows current, keep native transfer execution, content locations, and opaque recovery tokens host-owned, prove iOS background delivery on signed physical devices, and retain explicit interruption for public-cleartext or non-GET `WKDownload` fallbacks rather than replaying unsafe requests | B/C/D | Public preview |
| File uploads and pickers | Android document picker and iOS WebKit/user document-picker flows implemented without exposing paths to Vue; packaged Android/iOS cancellation and two-file selection prove OS-picker return delivery into the website | Complete signed-device platform-version coverage | C | Public preview |
| Site permissions | Shared `browser-core` exact-origin validation, camera/microphone/notification decision aggregation, OS-versus-site interpretation, and a host-keyed versioned repository now drive mobile; desktop consumes the same URL/origin validator while preserving its existing hostname-keyed store; Settings revocation, Android OS-grant and remembered-allow capture, packaged Android/iOS deny/restart/reset, and OS-versus-site denial evidence pass | Add signed physical-device allow evidence on both platforms | B/C | Public preview |
| Passwords and autofill | Platform-managed WebView/WKWebView Autofill is the supported baseline; a canonical non-secret Settings status is implemented and no Summer vault exists | Prove signed physical-device behavior and design any Summer vault separately | B/C | Public preview gate |
| Credit cards | Platform-managed browser-engine Autofill is the supported baseline; card data never enters Vue and the UI labels support unverified | Prove signed physical-device behavior before advertising support | B/C | Public preview gate |
| WebAuthn and passkeys | Non-secret capability status is implemented; Android browser-mode/provider approval and Apple managed-browser entitlement remain external gates | Implement and verify engine-owned behavior only after entitlements/provider policy are available | C | Public preview gate |
| Browser profiles | Not implemented | Explicitly outside the current blueprint | D | Deferred |
| Private browsing | Not implemented | Explicitly outside the current blueprint | D | Deferred |
| Summer App suggestions | Captain Word plus shared SDK provider execution implemented in both hosts; invalid providers, failures, timeouts, and stale requests are isolated, and matching packaged Android/iOS flows prove the provider row without History | Keep both packaged flows current, keep app-author `suggest(state)` portable, and retain cross-host orchestration fixtures as the contract evolves | A/B | Foundation-Daily MVP |
| Summer App pages | One reviewed manifest and TypeScript module supplies the Summer App Guide page, settings, and structured widget on Android/iOS; the same packaged Android/iOS flow changes its manifest-defined setting, proves the value in the responsive sandboxed page, reopens it through the structured widget, and confirms no History surface | Keep the same app entry and contract, add only responsive content in reviewed sandboxed pages, and retain matching packaged evidence on both hosts | B/C | Public preview-App parity |
| Summer App settings | The public `@summer/app-sdk/settings` entry now owns definition normalization, value/patch validation, defaults, text, and visibility policy; public manifest validation, desktop, and mobile consume it, so conformance reports the same field path either host rejects. Versioned ordinary settings, protected secret fields, live updates, reset, corrupt backup, and phone UI are implemented; Android uses Keystore AES-GCM, iOS uses a device-only non-synchronizing Keychain service, host snapshots expose only `configuredSecrets`, and matching Android/iOS packaged smoke proves save/restart/reset with no History; Android additionally audits logs for secret or Capacitor payload leakage | Keep both packaged flows current, and retain OS-vault separation and sanitized snapshots as the capability evolves | A/B/C | Apps parity |
| Summer App lifecycle | Mobile and desktop consume the public SDK context/module/lifecycle types and supply host/capability discovery; mobile serializes idempotent activation/deactivation with reverse-order cleanup, restores the native browser before deferred reviewed-app chunk activation, isolates per-app startup failures, and exposes only bounded app IDs plus trusted loading/partial-failure UI with coalesced in-place retry of failed modules that leaves working apps and tabs intact. Replaced catalog snapshots drive lifecycle-reactive settings and widgets; removal closes stale settings, cancels refresh timers, drops cached snapshots, and invalidates in-flight widget results. Desktop preserves its bounded storage, crypto, widget, tab, and exact-built-in facets | Keep cross-host lifecycle fixtures and browser-first packaged launch evidence current as new optional facets are added | A/B | Apps parity |
| Summer App compatibility | Publishable SDK build, manifest validator/schema, host reports, standalone examples, test host, and conformance CLI implemented; a disposable outside consumer installs the actual tarball, imports every public subpath including `@summer/app-sdk/settings`, type-checks against only shipped declarations, runs packaged conformance over a settings-bearing mobile manifest, and rejects private source/test leakage. Desktop package installation and mobile allowlisted-catalog admission call the same validator before code loading, including package-relative entry enforcement; repository discovery requires every reviewed manifest to pass it. Mobile then enforces host reports and catalogs compatible provider-only, page, and widget apps without importing disabled modules or fetching remote icons | Confirm license, registry ownership, final npm name, and support/versioning policy before publishing the package | A/B | Apps parity |
| Summer App installation | Compile-time allowlist only | Reviewed bundles first; signing and store-policy approval before downloaded apps | C/D | Apps parity-later |
| Widgets | Native structured cards remain supported; reviewed packaged apps may now reuse desktop sandboxed HTML/CSS/JavaScript for Library tiles, expanded overlays, and app-owned long-press menus. Apps own visibility requests, compact/expanded transitions, launcher imagery, actions, and update cadence through bounded contracts; Summer owns native circle geometry/drag persistence, website stacking, bridge validation, user presentation policy, and recovery | Keep packaged Android/iOS flows current; audit a separate native frame boundary before enabling downloaded widget code, and add reviewed OS home-screen adapters separately if desired | B/C/D | Apps parity |
| Browser imports/exports | Explicit user-selected Netscape bookmark HTML preview/import plus shared selection, duplicate/failure accounting, public-HTTP upgrade with loopback preservation, escaped serialization, and bounded native export implemented; one shared disposable fixture and flow provide focused packaged Android/iOS evidence for native picker delivery, explicit selection, 2/1/0 accounting, saved-row filtering, and fixture cleanup from the Bookmarks surface | Keep both packaged flows current; add other formats only as separate reviewed parsers/serializers | B/C | Apps parity |
| Find in page | One shared TypeScript policy validates bounded search/next/previous intents; mobile binds commands to the active tab, Android/iOS revalidate and translate them to engine calls, desktop validates renderer IPC and clipboard seeds through the same policy, and matching packaged Android/iOS flows prove success and no-match states | Keep both packaged flows current; add engine-error coverage only where the native engines expose deterministic hooks | B/C | Public preview |
| Media controls | Now Playing uses the same sandboxed HTML renderer on desktop and mobile, with bounded shared state/actions, revision-safe Android/iOS engine adapters, opaque app handles, browser-owned playback evidence, app-owned update cadence, `autoShow`/`alwaysOn` visibility, compact/expanded transitions, and durable per-session dismissal | Add richer metadata or source focusing only through separately bounded fields; never expose URLs, origins, native tab IDs, DOM access, or the Capacitor bridge | B/C | Apps parity |
| Context menus | Shared `browser-core` policy validates/canonicalizes HTTP(S) targets and enforces active-tab/revision-bound action eligibility; mobile uses it for open, new tab, native copy, and native share, while desktop uses the same target validation for Electron's open-in-new-tab action. Android unlinked images use the stale-safe mobile policy, iOS preserves WebKit's system image menu, and text selection remains engine-owned | Add richer image actions only through separately bounded contracts; keep native/Electron presentation host-owned and website scripts, selection text, and DOM access out of the trusted bridge | B/C/D | Apps parity |
| Popups and new windows | Native engines detect user gestures and forward only bounded raw routes; one shared `browser-core` policy requires the gesture, a still-live source tab, and a canonical credential-free HTTP(S) destination before opening an ordinary isolated tab. Privileged popup views are denied, and matching packaged Android/iOS popup-to-tab evidence passes | Keep both packaged flows current; expand only through the shared validated open-in-tab policy | B/C | Public preview |
| Notifications | Three bounded classes are implemented: generic opt-in download completion; declared Summer App status alerts with per-app authorization; and foreground-only website notifications for the active top-level exact origin after a page gesture and browser-owned site decision. Android/iOS revalidate at the native boundary, while website alerts accept only bounded title/body text and browser-owned origin provenance; matching packaged flows pass on both simulators | Keep both simulator flows current; keep web push, background delivery, actions, remote artwork, sensitive context, website URLs, and generic native options outside this slice | B/C | Public preview |
| Sharing | Shared TypeScript policy authorizes only an active credential-free HTTP(S) tab, then Android/iOS revalidate that the same tab and canonical URL are still active before opening their system share sheets; matching packaged Android/iOS cancellation and accessibility flows pass | Keep both packaged flows current | B/C | Public preview |
| Deep links | One shared `browser-core` gate parses direct HTTP(S) and exact `summer://open` routes, canonicalizes destinations, rejects unsafe/malformed input, and suppresses equivalent duplicate deliveries. Android/iOS forward only bounded raw routes; matching packaged Android/iOS flows prove cold launch, disposable blank-tab reuse, warm delivery, duplicate suppression, and invalid-route rejection | Keep release association metadata and both packaged flows current | B/C | Public preview |
| Default-browser integration | Android HTTP(S) BROWSABLE registration plus validated Android 10+ status/request UX and packaged system-consent cancellation evidence implemented; unsupported OS versions and the current iOS build report the capability unavailable | Complete the approved Apple entitlement, alternative browser engine, and release-registration workstream before enabling iOS selection | C/D | Apps parity |
| Cross-device sync | Not implemented | Explicitly outside the current blueprint; local models should not prevent later design | D | Deferred |
| Chrome extensions | Dynamic signed Chrome Web Store CRX3 inspection/install and a generic best-effort Android/iPhone runtime remain extension-ID-independent. Historical V51-V55 slices add exact reload/document authority/fairness, native same-extension Ports, foreground commands, a fail-closed DNR regex probe, and persistent storage access levels. Version 56 advances wrappers to schema v8, moves package backgrounds into a CSP-governed PAGE document with exact path carriers/readiness, and adds required-permission MV3 Promise-only `offscreen.createDocument`, `closeDocument`, and `hasDocument` for exact `WORKERS`: one hidden PAGE document per extension/eight globally, a provisional six-member runtime plus non-enumerable `lastError`, verified package modules/Workers, strict manifest-plus-Summer CSP/navigation/network boundaries, and process-only teardown. Final V56 Android packaged/real-extension, native/security, and Mac/Xcode/iPhone gates remain pending, so no V56 device or security-freeze claim is made. Cross-origin child broker authority, cross-extension/native-messaging Ports, other offscreen reasons, Chrome idle-worker/killed-app wake, many APIs, whole-extension claims, and live-iPhone behavior remain gaps | Keep local CRX/unpacked management desktop-only; evolve the clean-room API only through shared package contracts plus independently enforced Android/iOS native boundaries, and require exact packaged evidence before promoting each slice | B/C/D | Beta parity-later |
| Desktop windows, shortcuts, and menus | Not applicable | Replace with phone navigation and supported system integration | D | Redesign |
| Mesh identity and addresses | The exact bundled TypeScript Mesh app now supplies the same offline identity, rotating addresses, profiles, local suggestions, and future-network consent on desktop, Android, and iOS; mobile uses exact-bundle Android Keystore/iOS Keychain facets and durable native opt-in state, and the shared packaged flow proves disable/restart, profile recovery, and canonical address suggestion/open behavior on both simulators | Keep both simulator flows current and keep the app offline until the separately reviewed transport gates in [`mobile-mesh-design.md`](mobile-mesh-design.md) are satisfied | A/B/C | Apps parity |
| Mesh peer transport and `bar://` | Legacy desktop prototype remains disabled in normal startup | Keep outside the parity gates until authenticated signaling, transport, mobile lifecycle, privacy, and independent review requirements pass | B/C/D | Deferred |
| Updates and release | A fail-closed Android signed-AAB runner requires explicit versionCode plus environment-only key material, normally restricts publishable candidates to clean `master == origin/master`, and permits clean `develop == origin/develop` only through an explicit metadata-recorded override while master has an active release process. It verifies release manifest/version, JAR signer fingerprint, AGP-pinned bundletool format, bundled browser/App assets, hashes, and non-secret metadata. A separate dedicated-AVD rehearsal proves a same-key signed release upgrade and persisted browser state. A matching fail-closed iOS runner now transfers a bounded snapshot to the configured Mac, creates a signed Xcode archive and App Store Connect IPA, rejects version/bundle/team/profile-scope or asset mismatches, preserves the archive plus IPA with hashes and non-secret metadata, and never uploads; signed iOS device preparation remains separately automated | Create the first iOS App Store Connect record, complete signed physical-device evidence, produce a clean-source iOS candidate, complete store validation/upload/review/staged rollout, and prove upgrades from actual previous public versions using store-generated artifacts | C/D | Every release |

## Capability-specific notes

### Tabs, navigation, and restoration

The native engine owns live pages. TypeScript owns the logical tab state and
restoration policy.

A persisted tab record should include only stable logical data such as:

- tab ID and schema version;
- normalized URL;
- last known title and selected state;
- kept/pinned state if supported;
- logical ordering; and
- optional safe UI metadata.

Do not persist native view handles, process IDs, raw history objects, or bridge
callbacks in the portable record. Desktop still stores Electron
`RestoreOptions` privately so kept tabs retain their native navigation stack.
Its separate logical-session adapter projects ordered tab IDs, validated
current URLs/titles, active state, and kept state through the shared contract.
Internal or otherwise non-portable destinations become `null`; the adapter
never replaces or truncates the richer desktop restore record. On mobile
restore, validate records, create fresh native views, and tolerate individual
failures without discarding the rest of the session.

Native events need a monotonic tab revision. TypeScript ignores events older
than the latest accepted revision, which prevents a late navigation callback
from overwriting newer state after close, restore, or process recreation.

### Suggestions

Desktop and mobile now share suggestions at two deliberate boundaries. The
private `browser-core` matcher owns fuzzy matching, exact quick-key weighting,
group ordering, and app visibility for tab, bookmark, app, and provided rows.
The public SDK runner owns independent parallel provider execution, declaration
order, bounded timeouts, stale-query cancellation, output validation, failure
isolation, and query-scoped refresh coalescing.

Each host supplies normalized candidates and the current query, then applies
only host policy to the validated result. Mobile accepts HTTP(S) destinations;
desktop additionally attaches its app ID and permits `exclusive` only for a
trusted built-in app. Desktop and mobile remain free to render different rows,
keyboard behavior, and empty states without copying provider policy.

### Bookmarks and settings

Define repositories around domain operations, not storage primitives. Prefer
interfaces such as `listBookmarks`, `upsertBookmark`, and `removeBookmark` over
exposing generic key-value access.

Mobile history is intentionally absent. Do not add a History tab, store visit
records in the background, or add history-derived suggestions. Persisting the
validated URL of an open logical tab for crash/session restoration is session
state, not browsing history.

The legacy mobile bookmark copy is a migration seed. The shared repository now
writes and reads back the versioned replacement before removing that legacy
value, and retains a recoverable failure path.

Desktop keeps its existing bookmark store. Its adapter validates and projects
only the portable ID, name, HTTP(S) URL, quick keys, tags, and managed source.
Portable writes are read back for verification and merge into the existing
record, preserving desktop metadata, artwork, and unknown desktop-only fields.
Validation limits are host-configurable, so mobile bounds are not imposed on
desktop data.

### Downloads, uploads, and files

Each download is bound to its initiating tab, origin, URL, suggested filename,
and a stable download ID. TypeScript owns user-visible state; the native host
owns network/background behavior and the platform destination.

The contract must report start, progress, completion, cancellation, and failure
without exposing unrestricted filesystem paths. Upload and file-picker results
must contain only explicit user selections and must expire when the platform
access grant does.

Android snapshots include an opaque `android:<DownloadManager id>` recovery
token. On process recreation, the native adapter validates every persisted
candidate, queries only that app-owned job, and resumes bounded progress
reporting without exposing a destination path.

For iOS, an eligible HTTPS or local-network GET download is transferred from
`WKDownload` into an app-owned background `URLSession`. The adapter constructs a
new bounded request from an allowlist of WebKit request headers and matching
cookies, disables shared URLSession cookie storage, persists only validated
metadata, and exposes only an opaque `ios-background:<task id>` recovery token.
On relaunch it reconnects to that fixed session identifier, reconciles native
tasks, validates the final response URL, and moves the temporary payload inside
the download delegate callback. The packaged smoke models process loss with a
validated app-only `SIGKILL` and proves the same job completes after relaunch.

`WKDownload` remains the deliberate fallback for public cleartext HTTP and
non-GET requests. Those requests are not silently replayed in a different
network stack; if the process dies, their unfinished records become explicitly
`interrupted`. Neither path exposes a destination filesystem path to Vue.

A completed record has one trusted Library action: **Open or share**. TypeScript
sends only the stable opaque download ID and accepts only `presented` or
`unavailable`. Android revalidates completion, obtains a `DownloadManager`
content URI, grants temporary read access only to the system chooser, and never
returns that URI. iOS revalidates completion, resolves symlinks, requires a
regular file directly inside the app-owned Downloads directory, and passes only
that native URL to `UIActivityViewController`. Neither platform returns a path,
URI, or file handle through Capacitor.

### Permissions

Model two distinct layers:

1. Summer's site decision for an origin and capability; and
2. the operating system's permission state for the app.

A user can allow a site while the OS still denies the app. The UI and contract
must preserve this distinction so denial is actionable and revocation remains
accurate.

All prompts are bound to the active tab, exact top-level origin, and native
document lifetime. Starting a new top-level navigation, closing or deactivating
the tab, an origin mismatch, or timeout invalidates the request. Progress, title,
and other presentation callbacks may advance the tab snapshot revision without
invalidating the still-live native engine request.

Mobile stores only Summer's site-level camera and microphone decisions. A
blocked capability denies a combined request; all requested capabilities must
be allowed before Summer can skip its own prompt. Returning both capabilities
to `ask` removes the origin record. The operating system remains authoritative
for hardware access, so an allowed site can still show an OS-blocked state.
When that happens, Summer preserves the site-level Allow, expands trusted chrome
with actionable device-setting feedback, and leaves the website engine's denied
result intact. The packaged Android flow proves that distinction and Settings
state without granting hardware access.

### Passwords, cards, autofill, and passkeys

TypeScript may own capability screens, non-secret state, validation, and
policy. Secret values and credential handles remain inside the website engine,
operating-system provider, Keychain, Keystore, or a separately approved
platform vault.

A website never queries the trusted Vue store for credentials. In the current
platform-managed design, Autofill and WebAuthentication complete between the
isolated native website view and the operating system; the Capacitor adapter
returns only canonical capability states. Logs and errors contain no origin,
account, provider, credential ID, password, assertion, or card data.

Cross-device credential synchronization is outside the current blueprint. If it is
reconsidered later, it requires a separate design for end-to-end encryption,
device enrollment, recovery, revocation, export, and data loss.

### Deferred: profiles and private browsing

Multiple profiles and private browsing are explicitly outside the current mobile
blueprint. They should not add work to the MVP or Public preview acceptance gates.

Current repository and host contracts should still avoid unnecessary global
singletons or irreversible assumptions that would make future isolation
impossible. This is architectural hygiene, not a commitment to implement either
feature. If they return to scope, they need a separate design for tab restore,
suggestions, downloads, credentials, Summer Apps, crash recovery, and
platform website-data isolation.

### Find, media, menus, and popups

Reuse command policy and state, not desktop presentation. Phones need sheets,
touch menus, compact find controls, and clear back behavior.

Find uses one `browser-core` action vocabulary: search, next, and previous. The
shared policy preserves meaningful query whitespace, caps input at 512 UTF-16
code units, and binds phone commands to an active tab. Android, iOS, and Electron
retain their engine-specific execution semantics and revalidate at their native
or privileged boundary; desktop renderer IPC and clipboard seeds pass through
the same intent gate.

Long-pressed link detection stays in each isolated website engine. The native
host emits only a bounded HTTP(S) URL plus the active tab ID and revision; the
shared `browser-core` policy canonicalizes the target, checks action eligibility,
and rejects stale tab identities before the phone offers open, open in new tab,
native copy, or native share. Navigation invalidates the request. Desktop keeps
its Electron menu but routes open-in-new-tab targets through the same validator.

Android reports an unlinked image through a separate event containing only a
validated HTTP(S) image address plus the active tab ID and revision. The trusted
Vue sheet can open, open in a new tab, copy, or share that address, and becomes
inert as soon as its originating navigation is stale. iOS WebKit does not expose
an image URL through its public context-menu element contract, so Summer leaves
the system image menu intact for sharing, saving, and copying rather than
injecting a website script or message bridge.

Text selection stays engine-owned on both platforms. Summer does not extract
selected text, install a page script, or add selection data to the Capacitor
contract. Any future app-visible selection feature requires its own bounded
payload, origin policy, and privacy review rather than expanding either link or
image events.

Popup handling is a security boundary. Android and iOS keep gesture detection in
the website engine and forward only a bounded raw route plus source-tab identity.
Shared `browser-core` policy requires that engine-confirmed gesture, rejects a
request after its source tab closes, and canonicalizes only credential-free
HTTP(S) destinations before opening an ordinary isolated tab. The temporary
Android capture view has no JavaScript, storage, file/content access, or bridge;
iOS returns no popup view. Neither host creates a view with broader access than
the opener.

### Notifications, sharing, and deep links

Current-page sharing begins in shared TypeScript policy, which accepts only the
active tab and canonicalizes a credential-free HTTP(S) URL. Android and iOS then
confirm that the same native tab and canonical URL are still active immediately
before presenting their OS-owned share sheet. A tab switch or navigation makes
the command inert. Link and Android image-target sharing use the separate shared
context-target policy described above; neither path exposes website content or a
generic native sharing port to Summer Apps.

Notification delivery has two deliberately separate policy classes. The
download-completion slice starts from an explicit trusted Settings opt-in.
TypeScript owns a bounded non-interactive intent containing only an ID, title,
and body; Android and iOS own permission and OS delivery. Completion alerts are
deduplicated and deliberately omit website, origin, account, URL, and filename
data.

A Summer App may separately declare `capabilities: ["notifications"]` and
feature-detect `context.notifications`. The app supplies only an app-local ID,
title, and body. The browser derives the app ID and display name from its
reviewed definition, and the mobile host requires a browser-owned per-app Apps
Library opt-in plus OS permission. Android and iOS independently revalidate both,
scope native notification identifiers to the app, and expose no URL or action.
The host-neutral Guide app exercises the same TypeScript request on desktop,
Android, and iOS.

A hostile local website fixture probes the privileged Capacitor, generic
Android/iOS bridge, Summer, and Electron global names from inside an isolated
website tab. Static Android/iOS contract tests bind that fixture to their
absence and separately require the sole fixed notification channel to enforce
main-frame, exact-origin, active-tab, foreground, gesture, and bounded-payload
checks. The bridge-isolation packaged flow passes on Android and iOS and asserts
that no History surface appears.

Websites receive a separate, deliberately narrow `Notification` compatibility
surface only while the active top-level exact origin is in the foreground. A
page gesture may request the browser-owned site decision, and an allowed origin
may submit only bounded title/body text. Native code revalidates the main frame,
source origin, active tab, foreground state, remembered decision, and OS
permission before delivery. There is no web push, service worker, background
delivery, URL, action, remote artwork, or generic native-option port; see
[`mobile-website-notifications-design.md`](mobile-website-notifications-design.md).

Android and iOS forward only a bounded raw external route from their OS lifecycle
hooks. One shared `browser-core` gate parses direct HTTP(S) input or the exact
`summer://open?url=...` shape, canonicalizes and validates the destination, and
suppresses duplicate delivery IDs or equivalent target URLs before creating a
tab. The cross-platform packaged flow covers a cold destination launch that
reuses a sole disposable blank session tab, a distinct warm delivery, immediate
duplicate suppression, and an invalid `javascript:` payload without adding
History. It passes on packaged Android and iOS.

### Dynamic Chrome WebExtensions

Mobile does not receive Chromium's extension engine from Android WebView or
WKWebView. Summer therefore treats the shared Extensions Manager as review and
lifecycle UI, shared TypeScript as the package/compiler contract, and Java/Kotlin
or Swift as independent privileged hosts. Any signed Chrome Web Store CRX3 may
be inspected and installed; compatibility is determined by the generic API and
manifest subset, never by an extension ID or package-specific branch. Local CRX
and unpacked installation remain desktop-only.

Version 56's new public slice is the Manifest V3 required-permission
`chrome.offscreen` / `browser.offscreen` API. Only an authenticated current
background or top-level extension-owned page receives Promise-only
`createDocument({url, reasons: ["WORKERS"], justification})`,
`closeDocument()`, and `hasDocument()`. The exact one-element `WORKERS` reason
is the only admitted reason; optional-only declarations, callbacks, ordinary
content/page callers, and calls from the hidden document itself fail closed.
The canonical URL must select a verified same-extension packaged HTML resource,
with no credentials, port, traversal/ambiguity, or cross-extension host.

Native quota is one creating, active, or closing document per extension and
eight globally. The 25-second creation transaction resolves only after the
immutable initial main frame and provisional runtime are both ready, allowing an external
module to use top-level-await runtime messaging before the caller settles. The
hidden PAGE document receives only `runtime.id`, `getURL`, `sendMessage`,
`connect`, `onMessage`, `onConnect`, and non-enumerable `lastError`. Its modules,
Workers, WASM, and other resources come from an exact view/extension/document/
generation-bound package responder. The sanitized manifest extension-pages CSP
is intersected with a package-only Summer script/Worker floor; inline/remote
scripts, direct connections, frames, navigation, popups, forms, downloads, and
arbitrary base/object destinations remain denied. Eligible HTTP(S) fetch/XHR
still uses only the existing cookie-free broker after current host-permission,
manifest-CSP, and per-redirect checks.

Creation belongs to the extension after native acceptance, even when its caller
disappears. Exact state may survive background replacement, unrelated
configuration changes, and an unchanged exact-compatible owning-extension
configuration. Explicit close/`window.close()`, the owning
extension's reload/update/disable/removal, incompatible reconfiguration, load or
navigation failure, renderer/process loss, and browser exit destroy the view,
Ports, pending operations, and quota. This is process-only state and supplies no
idle-worker, suspended-app, killed-app, audio, or general background-execution
guarantee. Schema v8 is mandatory; older generated sources remain visible but
disabled until the signed package is reinstalled and reparsed.

V56 final Android packaged and real-extension smokes, post-fix native/security
review, Mac/Xcode Simulator evidence, and live-iPhone behavior remain pending.
Shared or native source contracts are not substitutes for those runtime gates.

Version 51's new public slice is `chrome.runtime.reload()` /
`browser.runtime.reload()` in backgrounds, extension pages, and authenticated
content scripts. It accepts no arguments and returns `undefined` synchronously.
The native host accepts only the exact three-key inner request, acknowledges it
before destructive work, and schedules a target-only replacement. Normal reload
clears the target's `storage.session` and queued session-change events, advances
its runtime epoch, invalidates transient document/reply/network/scripting work,
and rebuilds its hidden background and owned pages. It preserves siblings,
ordinary website tabs, durable local/sync storage, permission decisions, alarms,
menus, static/dynamic/session DNR rules, and eligible non-session background
events.

Both hosts enforce five accepted reloads per rolling ten seconds. The sixth is
acknowledged, then terminates and suppresses the exact installed target until an
explicit manager configuration/re-enable or replacement identity. A queued
event or renderer-recovery callback cannot revive it. Asynchronous DNR,
scripting, storage, and page work carries the target epoch/source so pre-reload
completion cannot win an ABA race after the replacement.

Version 51 persisted generated wrapper source with schema v2. Records older than
the current schema stay visible but disabled as load-failed and must be
reinstalled and reparsed. Do not migrate them by changing the stored version
number: the signed package must pass the current compiler and review path.

Document authority is established natively before package code. Shared READY
registration has the exact fields `version`, `type`, `documentToken`,
`timeOrigin`, and `topTimeOrigin`. iPhone direct runtime/storage requests use an
exact outer `context.request` with `version`, `type`, `documentToken`, and
`request`; Swift accepts the inner operation only while that token identifies
the current background generation, popup, owned page, or verified ordinary top
document. Android uses the corresponding ordered native-world bootstrap,
secret/token registration, and current receiver/proxy/top-document proof. Never
fall back to URL/origin equality, an engine object retained from an earlier
navigation, or a JavaScript `userGesture` Boolean as authority.

Ordinary website broker authority remains top-frame-only. The static matching
layer may inject supported child/related-frame declarations, but cross-origin
child runtime/storage calls fail closed until both hosts have a durable,
navigation-safe child-document proof. Record this limitation separately from
static script injection coverage.

Global ceilings must leave capacity for another installed extension. Version 51
pairs each relevant shared pool with an extension-scoped admission limit; the
streamed DNR and user-script read pools, for example, allow one extension no
more than two of eight global slots. Admission, end, timeout, reconfiguration,
and source-loss paths must reserve and release the same accounting. Apply this
pattern to every new shared queue or retained-byte pool rather than adding only
a global cap.

Version 51's focused gate passes Android native contracts 73/73, iOS native
source contracts 63/63, shared package/migration contracts 79/79, the JDK
compile, and the connected-Mac iOS build-only gate. Its signed Android Dark
Reader flow passed the complete tested lifecycle including cold restart in 217
seconds. The signed uBlock Origin Lite flow passed its tested static block in
140.6 seconds and recovered after `runtime.reload()`. These prove only the
listed behavior, not full package compatibility or live iPhone behavior.

Version 52 persists only schema-v3 generated wrappers. Every older generated
record remains visible but disabled until the signed package is reinstalled and
reparsed. Its same-extension Port registry is native-owned. Two opaque endpoint
IDs identify the connection sides but grant no authority; source document,
extension version, runtime epoch, and background generation must remain exact.
OPEN queues CONNECT before publishing success and ACTIVATE begins delivery.
Messages are FIFO; local disconnect is silent locally and reaches the peer once;
endpoint loss, reload, navigation, or renderer crash reaches the exact surviving
endpoint once. Post-close sends throw, and `runtime.lastError` is scoped to the
disconnect listener call that observes it.

Port admission is limited to 64 per extension and 256 globally. Queues allow 64
messages and 256 KiB per Port, 256 messages and 1 MiB per extension, and 2,048
messages and 8 MiB globally for at most 30 seconds. Cross-extension Ports and
`connectNative()` remain absent. A
pre-reload content context cannot be adopted by the replacement generation and
must navigate to obtain fresh authority. The existing top-frame-only website
boundary and no-idle-worker/no-killed-app-wake limitations remain.

The final V52 focused gates pass: shared Port contracts 91/91, Android Port
contracts 76/76 plus JDK compilation, iOS Port contracts 69/69, exact
content-scope contracts 115/115 shared, 78/78 Android plus JDK compilation,
70/70 iOS, and 272/272 integrated, plus the 41-file/486-test package gate,
TypeScript check, and package diff check. Security reviews returned FREEZE/no
P1/P2 for shared/Android/iOS Ports, exact content scopes, and the viewport
transition. A fresh 94.7-second remote-Mac gate passed the production
bundle/typecheck, Capacitor iOS copy, Xcode iOS Simulator build, and simulator
installation.

The final debug APK at
`packages/summer-android/android/app/build/outputs/apk/debug/app-debug.apk` is
24,963,237 bytes, was built at 2026-08-13 05:01:02 UTC, has SHA-256
`7625131D6B4AC38533E94375619B86C06ED83BFCB9460ED58B3661EDD38914BA`, and was
installed on Android 16 `emulator-5554`. On that exact APK, signed Dark Reader
passed install, active content behavior, cold restart, disable, re-enable, and
removal in 246.5 seconds; the deterministic signed Port fixture passed ordered echo,
pre-open FIFO, local disconnect, background-reload replacement, and navigation
teardown in 175.1 seconds; and signed uBlock Origin Lite passed install and the
visible `Network advertisement blocked` assertion in 130.8 seconds. Do not relabel these focused packaged
Android and simulator/source-contract results as whole-extension, live-iPhone,
or cross-platform parity.

## Data ownership and current scope

Desktop storage implementations must not become mobile interchange formats.
Use versioned logical repositories even though cross-device synchronization is
not currently planned.

| Data | Shared contract | Desktop adapter | Mobile adapter | Current position |
| --- | --- | --- | --- | --- |
| Settings | Validated versioned object plus explicit host projection | Existing stores behind a semantics-preserving adapter | Versioned redundant bounded storage | Local-only |
| Bookmarks | Stable IDs, URL, title, tags, and quick keys | Existing store behind a lossless portable projection | Versioned redundant bounded storage | Local-only; exportable |
| History | Not part of the mobile product | Desktop history remains desktop-owned | No mobile visit store | Excluded by product decision |
| Tab sessions | Validated snapshots, host-configurable bounds, and restore policy | Existing native-history restore plus a separate logical projection | Redundant app snapshots plus native recreation | Local-only |
| Passwords/cards | Canonical capability state only | Existing credential service | Platform-managed WebView/WKWebView Autofill; no Summer vault | Platform-owned; signed-device verification pending |
| Cookies/site data | Shared cookie/cache-only clear request | Electron sessions | Implemented Android `CookieManager`/WebView cache and iOS `WKWebsiteDataStore` adapters | Platform-owned; never copied |
| Summer App settings | Defaults, overrides, and schema | Existing app settings service | App-owned repository | Local-only |

Every repository should define:

- schema version and runtime validation;
- atomic migrations and rollback/recovery behavior;
- stable record IDs and timestamps;
- deletion semantics and export format; and
- corrupt-record handling.

There is no current workstream for accounts, devices, network replication,
conflict resolution, or synchronized deletion. If sync is proposed later, it
must receive its own product and security design. Raw cookies, cache,
browser-engine folders, and platform credential handles must never become sync
records.

## Summer Apps transfer

### Goal for app authors

A TypeScript Summer App should not become a separate Android app and iOS app.
The normal path is one app package and one runtime implementation across Summer
hosts.

Every checked-in first-party, SDK-example, and end-to-end fixture manifest now
carries an explicit six-key platform claim. The repository compatibility test
discovers those manifests, requires each to pass the public SDK validator, and
fails when a new one is not added to the reviewed matrix. Extensions Manager
remains desktop-only. Mesh, Captain Word, Media
Player, and the SDK's portable Summer App Guide are mobile-eligible reviewed
built-ins loaded by the phone host; Mesh is explicitly user-enabled and receives
its credential facet only as the exact compile-time registry entry.

For an existing host-neutral app, phone support should usually require only:

1. use the public SDK instead of browser-private imports;
2. set all six existing `platforms` booleans accurately, enabling `android` and
   `ios` when tested or `any` only when every named host is genuinely supported;
3. keep declared roles and manifest capabilities within what each host reports
   as supported;
4. remove or isolate Electron/Node.js assumptions;
5. make an app page responsive and touch-friendly when it has UI; and
6. pass the SDK conformance suite on desktop, Android, and iOS.

Suggestion-only apps such as Captain Word are the easiest case: host-neutral
TypeScript logic and structured results can normally be shared directly. Page,
widget, file, notification, or other privileged integrations require more host
capabilities, but should still use one public contract rather than a separate
mobile application.

### Compatibility model

Summer already has useful explicit declarations. Keep them instead of adding a
second mobile-only manifest:

| Existing manifest signal | Capability meaning |
| --- | --- |
| `platforms` | Host eligibility for `windows`, `ios`, `macos`, `linux`, `android`, and `any` |
| `type` | App roles such as `SummerApp`, `SuggestionProvider`, `WidgetProvider`, and `RawTextEngine` |
| `settings` | Requires normalized setting defaults; declaring any `secret` field additionally infers the `settings-secrets` host capability |
| `widgets` | Requires declared widget presentation and action capabilities |
| `capabilities` | Declares an additional host API requirement such as `media-control` when it cannot be inferred from a role |
| `customProtocols` | Requires protocol registration/dispatch support |
| `permissions` | Discloses privileged behavior for review; it is not currently an enforced sandbox |

The public SDK should normalize these declarations into an explicit per-host
compatibility report:

```json
{
  "host": "ios",
  "eligible": true,
  "supported": ["suggestions", "settings-defaults"],
  "blocked": []
}
```

The compatibility baseline deliberately separates `settings-defaults` from
`settings-secrets`. The first proves that a host can validate the schema and
expose declared defaults. A manifest with any `secret` field also requires the
inferred `settings-secrets` capability. Desktop advertises that capability for
its existing protected store. Android and iOS advertise it only when the native
vault adapter is present: ordinary overrides stay in bounded app storage,
secret values use a separate OS vault, the owning app runtime receives the
value, and every host-facing snapshot receives only `configuredSecrets`.

Capability-bearing fields fail closed when present with the wrong JSON shape.
For example, scalar `customProtocols`, array `permissions`, or object `widgets`
values are manifest errors, not absent requirements.
The same public validator authorizes desktop package installation and mobile
allowlisted-catalog admission before either host loads app code. The `main`
entry must remain package-relative and cannot contain parent traversal, absolute
paths, empty segments, or platform-reserved path characters.

Eligibility alone is insufficient. An app can claim `platforms.ios: true` but
still be blocked because its role, entry point, or required host capability is
unsupported. Diagnostics must name the exact missing capability and remediation.

Do not require authors to repeat roles, settings, widgets, or protocols in a new
mobile list. If a future API dependency cannot be inferred from existing
manifest fields, add one host-neutral, versioned requirement declaration through
the SDK rather than Android- and iOS-specific copies.

The Now Playing transfer demonstrates the exception for form-factor-specific
presentation without creating a second app package. Its existing desktop
`renderer.type: "sandboxed"` stays intact, while `mobileRenderer.type: "native"`
selects a reviewed structured snapshot renderer on Android and iOS. The module
uses the optional `context.media` facet only after declaring
`capabilities: ["media-control"]`. Session IDs are opaque and revision-bound;
the app receives title, paused state, bounded time/duration, and seek support,
but no URL, origin, native tab ID, artwork, DOM object, or website bridge.

### Public SDK boundary

The foundation repository path is `packages/summer-app-sdk`. Its published npm
scope and name are deliberately undecided until registry ownership and support
policy are chosen. The package should contain only stable app-author contracts:

- manifest types, schema, and runtime validation;
- platform and capability evaluation;
- lifecycle, suggestions, pages, and structured-widget types plus complete
  portable settings normalization, validation, defaults, text, and
  visibility policy;
- host-neutral `SummerAppContext` interfaces;
- structured error and cancellation behavior;
- fixtures and fake hosts for unit tests;
- a conformance CLI that prints compatibility per Summer host; and
- migration notes and deprecation diagnostics between SDK versions.

The package should be independently versioned and publishable from the monorepo.
It must be browser-safe, documented with runnable TypeScript examples, and usable
without cloning Summer. A starter template can depend on it, but the SDK must
also support existing manifest-plus-entry packages so adoption is incremental.

Keep browser runtime implementations, Electron/Capacitor adapters, local stores,
review allowlists, signing keys, and release credentials out of the public SDK.
A public type is not permission to expose a privileged operation.

### SDK extraction sequence

Do not start by moving every app type. Extract the safe boundary in small,
compatibility-preserving changes:

1. Implemented: `packages/summer-app-sdk` is an explicit root workspace with
   compiled JavaScript/declarations, typechecks, focused tests, and a built-entry
   import smoke check.
2. Implemented: the SDK owns platform types, explicit six-platform policy, and
   host capability reporting while compatibility re-exports preserve callers.
3. Implemented: pure SDK modules own portable manifests, the complete settings
   policy, suggestion/page/widget results, and lifecycle contracts. A
   compatibility re-export preserves internal callers while mobile imports the public settings
   subpath directly; the desktop type barrel retains only host-specific records.
4. Implemented: mobile consumes the SDK context/module contract and projects
   only a host-owned normalized subset for its runtime; both mobile catalog
   admission and desktop package installation authorize untrusted manifests
   through the same SDK validator before code loading.
5. Implemented: the SDK owns the portable JSON Schema object while the root
   `sap.schema.json` compatibility path remains contract-tested.
6. Implemented: per-host diagnostics, fake-host fixtures, a conformance CLI,
   and host-neutral suggestion and page/settings/widget examples are
   independently built and tested.

Each step must keep the checked-in manifest schema, TypeScript types, runtime
validation, documentation, and tests aligned. Compatibility re-exports can be
removed only through a separately documented migration.

### Shared runtime behavior

Extract and share through the product core and SDK boundary:

- manifest and platform validation;
- lifecycle states and error classification;
- settings manifest normalization, value and patch validation, defaults, text,
  visibility, and bounded host updates;
- suggestion-provider cancellation, timeout, and result validation;
- page response type and size validation;
- structured widget snapshot/action validation; and
- activation/deactivation test vectors.

Keep platform-specific:

- bundle discovery, signature verification, installation, and update policy;
- secure storage and app-owned data locations;
- page and widget presentation;
- notifications, sharing, and file capabilities; and
- store-policy enforcement.

### Initial security and distribution position

The first mobile releases should continue to load reviewed, packaged TypeScript
from an explicit allowlist. Bundled code runs in the trusted application and is
therefore a security decision even when its output is validated.

Downloaded Summer Apps remain a later capability. They require an approved
signing, review, revocation, update, and store-compliance model. Making app
authoring easy does not require immediately allowing unreviewed executable code.

App pages remain separate from website views. An app page receives only declared,
bounded capabilities; a website receives none of the trusted app bridge.

### Widget adaptation

Reviewed packaged widgets can reuse their desktop sandboxed HTML, content, and
action contracts while adapting to the mobile host's `tile`, `expanded`, and
`menu` presentation context. Native structured cards remain available. Summer
owns overlay safety, circle drag geometry, and the bounded bridge; the app owns
its rendered pixels, behavior, presentation requests, and refresh cadence.
Downloaded custom code still requires a separate signed installation and native
frame-isolation review and is not implied by packaged-widget support.

## File-level change map

| Area | Expected change |
| --- | --- |
| [`package.json`](../package.json) | Now lists both foundation packages and their focused tests; add future shared packages explicitly |
| `packages/browser-core` | Owns suggestion and search-query policy, shared tab snapshots/reducer, host-configurable bookmark/session/download validation and repository behavior, download recovery/reconciliation policy, and bookmark-import selection/deduplication/URL policy; grow it with additional internal models, migrations, and fixtures |
| `packages/summer-app-sdk` | Owns pure portable type modules, the complete settings policy, runtime manifest validation, host compatibility, fake hosts, conformance CLI, examples, compiled output, and version policy |
| [`types/summerApps.d.ts`](../types/summerApps.d.ts) | Compatibility-reexports pure SDK types; retains only Electron installed-app records and suggestion IPC routing metadata |
| [`sap.schema.json`](../sap.schema.json) | Remain a compatibility path generated or checked against the SDK-owned portable schema |
| [`shared`](../shared) | Migration source for current pure policy such as platform validation; retain repository-wide contracts not yet owned by a package |
| [`ui/store`](../ui/store) | Suggestion cards now adapt to browser core; continue separating reusable behavior from Electron IPC, persistence, and singleton state |
| [`ui/store/desktopBrowserSettingsAdapter.ts`](../ui/store/desktopBrowserSettingsAdapter.ts) | Projects matching desktop theme, locale, and search values into the shared settings contract; only portable theme writes are supported, and custom themes/providers remain untouched |
| [`ui/components`](../ui/components) | Extract form-factor-neutral controls and headless behavior; retain desktop composition |
| [`electron/main/front`](../electron/main/front) | The observational tab adapter projects Electron state into shared snapshots without replacing desktop behavior; continue implementing desktop hosts for sessions, downloads, permissions, and media |
| [`electron/main/desktopBookmarkAdapter.ts`](../electron/main/desktopBookmarkAdapter.ts) | Losslessly projects the existing desktop bookmark store into portable records and preserves desktop-only fields on verified writes |
| [`electron/main/desktopTabSessionAdapter.ts`](../electron/main/desktopTabSessionAdapter.ts) | Projects live desktop tabs into logical session records while keeping Electron navigation histories private and untouched |
| [`electron/main/apps`](../electron/main/apps) | Consume shared SDK/core policy; retain desktop installation, privileged context, and presentation |
| [`electron/preload`](../electron/preload) | Expose only capability-specific desktop adapters |
| [`packages/summer-android/src/domain`](../packages/summer-android/src/domain) | Move mature pure behavior into browser core; retain mobile composition |
| [`mobileSummerAppRuntime.ts`](../packages/summer-android/src/summerApps/mobileSummerAppRuntime.ts) | Enforces the SDK compatibility report and consumes the public context/module/lifecycle contracts while retaining mobile activation and presentation |
| [`packages/summer-android/src/native`](../packages/summer-android/src/native) | Grow runtime-validated host contracts and Capacitor adapters |
| [`SummerTabsPlugin.java`](../packages/summer-android/android/app/src/main/java/com/hiketech/summerbrowser/SummerTabsPlugin.java) | Implement Android engine/system capabilities without a generic bridge |
| [`SummerTabsPlugin.swift`](../packages/summer-android/ios/App/App/SummerTabsPlugin.swift) | Implement matching iOS capabilities with isolated `WKWebView` configurations |
| [`packages/summer-android/src/summerApps`](../packages/summer-android/src/summerApps) | Retain reviewed mobile allowlist and platform presentation |
| [`examples`](../examples) | Add one host-neutral app and intentional capability-failure fixtures for SDK conformance |
| [`docs/summer-app-development.md`](summer-app-development.md) | Make desktop/Android/iOS authoring and compatibility diagnostics one documented workflow |
| [`tests`](../tests) | Domain, migration, SDK, contract, security, and desktop-adapter tests |
| [`packages/summer-android/.maestro`](../packages/summer-android/.maestro) | Android/iOS packaged smoke coverage for completed workflows |

The `summer-android` name is historically inaccurate because the package now
supports Android and iOS. Renaming it to `summer-mobile` is reasonable after
scripts and imports stabilize, but it unlocks no functionality and should not be
mixed into behavior extraction.

## Parity evidence for each slice

A capability may be implemented on one host first, but it remains guarded and
incomplete until every required row is satisfied.

| Evidence area | Required proof |
| --- | --- |
| Shared behavior | Deterministic fixtures cover current policy, invalid input, limits, stale data, and migration where applicable |
| Desktop | Existing Electron behavior remains covered after extraction |
| Android | Java/Kotlin adapter and packaged workflow cover success plus relevant denial/failure/recovery paths |
| iOS | Swift adapter covers the same contract and workflow, including `WKWebView` process recovery where relevant |
| Vue UX | Loading, empty, progress, cancellation, denial, error, safe-area, keyboard, narrow, tablet, and RTL states as applicable |
| Security | Origin/tab binding, bounded bridge values, malicious callbacks, and website-to-bridge isolation |
| Documentation | Baseline, capability row, app compatibility, commands, and known exclusions match delivered behavior |

A feature flag may hide an incomplete host implementation. Documentation must
still say which side is missing; a shared TypeScript build is not native parity.

## Risks and controls

| Risk | Control |
| --- | --- |
| Desktop and mobile behavior drift | One pure policy implementation and shared fixtures |
| A universal bridge becomes a privilege escape | Capability-specific contracts, origin binding, revisions, and bounded payloads |
| Website content reaches Capacitor | Separate website views and packaged bridge-isolation tests |
| Android and iOS native behavior diverges | One contract, matching test vectors, and explicit parity review |
| Storage migration loses data | Versioned schemas, atomic changes, upgrade tests, export, and recovery |
| Credentials enter renderer state or logs | Native vaults, non-secret summaries, redaction, and origin-scoped operations |
| Desktop UX is unusable on phones | Preserve behavior, redesign interaction, and test narrow/RTL/keyboard states |
| App authors must maintain mobile forks | One SDK, one manifest model, capability diagnostics, and host-neutral examples |
| Public SDK churn breaks outside apps | Independent versioning, compatibility fixtures, deprecation diagnostics, and migration notes |
| Store policy blocks executable updates | Reviewed bundled apps until a compliant signed model is approved |
| Deferred scope quietly becomes MVP work | Keep sync, profiles, private browsing, and P2P behind explicit later decisions |
| P2P harms battery or background behavior | Keep it post-parity and require real-device evidence |

## Verification command map

Commands prove different layers. Do not report one as another.

| Layer | Current command | What it proves |
| --- | --- | --- |
| Targeted desktop/shared unit | `npm run test:unit -- tests/<name>.test.ts` | Selected Vitest behavior and contract fixtures |
| Shared browser policy | `npm run verify --workspace @summer/browser-core` | Host-neutral policy typecheck and deterministic fixtures, including strict pathless download projection, persistence failure, corruption recovery, and process-loss reconciliation |
| Public SDK package | `npm run verify --workspace @summer/app-sdk` | SDK types/examples/tests plus built entry points and an actual tarball installed, imported, type-checked, and conformance-checked by a disposable outside consumer |
| Desktop TypeScript/Vue | `npm run build:vue` | Desktop typecheck and renderer build |
| Mobile unit tests | `npm run test:android` or `npm run test:ios` | The same mobile Vitest suite; names select the workspace, not a native platform |
| Mobile TypeScript/Vue | `npm run build:android` or `npm run build:ios` | The same mobile typecheck and Vite build; this is not a Gradle/Xcode build |
| Android asset sync | `npm run android:sync` | Mobile web build plus Capacitor synchronization into the Android project |
| iOS asset sync | `npm run ios:sync` | Mobile web build plus Capacitor synchronization into the iOS project; run where the iOS toolchain is available |
| Android packaged smoke | `npm run test:smoke:android` | Maestro workflow against an available Android emulator/device |
| Shared browser-first launch smoke | Android: `npm run test:smoke:android -- --flow launch-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow launch-smoke.yaml` | The shared flow observes browser `Ready` before entering the Apps Library, waits for the deferred Summer App Guide catalog, and proves no History surface. It passes in the complete Android and iOS simulator suites |
| Shared reviewed-app catalog smoke | Android: `npm run test:smoke:android -- --flow summer-app-catalog-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow summer-app-catalog-smoke.yaml` | The shared flow proves CaptainWord's suggestion-only role, the Guide page/widget roles, disabled Mesh page/suggestion roles, Now Playing's widget-only role, and no History surface. On 2026-08-03 it passed on `Medium_Phone_API_35` and the iPhone 17 Pro / iOS 26.5 simulator against the exact bundle that moved both hosts onto the shared SDK manifest validator |
| Shared Settings persistence and RTL smoke | Android: `npm run test:smoke:android -- --flow settings-persistence-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow settings-persistence-smoke.yaml` | The identical packaged flow changes system theme to dark, system language to Hebrew, and Google to DuckDuckGo, observes the localized current values and Hebrew address chrome, stops and relaunches the app without clearing state, and proves all three choices remain while History stays absent. On 2026-08-03 it passed on `Medium_Phone_API_35` and the iPhone 17 Pro / iOS 26.5 simulator |
| Current Android complete flow-set proof (2026-08-03) | Aggregate: `npm run test:smoke:android`; focused recovery: `library-smoke.yaml`, `tablet-library-smoke.yaml`, and `permission-os-denial-android-smoke.yaml` through their documented runners | The current debug APK completed a 33m 11s aggregate with 34/37 flows passing on the warmed software-rendered `Medium_Phone_API_35` emulator. Two failures were stale bare-label assertions after Settings gained stateful accessible names; the third exposed two visually identical Android permission dialogs without a verified camera-to-microphone transition. After aligning the Library flows and making each OS denial target the permission-controller button with retry-on-no-change plus an explicit audio-prompt wait, phone Library passed in 1m 09s, the reversible tablet-dimension flow passed in 1m 21s, and the complete denial/Settings/no-History flow passed in 2m 12s. Together these runs cover all 37 current flows; this is complete current flow coverage, not a single 37/37 final-harness aggregate |
| Current iOS complete workflow proof (2026-08-03) | `npm run test:smoke:ios:remote -- --skip-build` | One uninterrupted remote workflow on the iPhone 17 Pro / iOS 26.5 simulator passed all 34 standard flows in 13m 55s, then passed all six custom notification, website-notification, release-upgrade, and forced-process-loss download-recovery phases; the complete command finished in 19m 35s. The evidence covers Mesh, tabs, bookmark import/export, downloads, Apps, Settings persistence and RTL, permissions, deep links, sharing, session restore, and explicit no-History checks. It is simulator evidence, not signed physical-device or App Store evidence |
| Android download-recovery proof (2026-08-02) | `npm run test:smoke:android -- --flow download-recovery-smoke.yaml` | The rebuilt debug APK retained `slow-download.bin` across force-stop/relaunch through the shared recovery-candidate and reconciliation policy, did not label it interrupted, and observed completion on `emulator-5554` in 2m 06s; cleanup left zero disposable picker fixtures and zero reverse mappings |
| Android completed-download presentation proof (2026-08-02) | `npm run test:smoke:android -- --flow download-presentation-smoke.yaml` | A real local fixture reached Completed, the trusted Library's **Open or share** action opened Android's **Sharing 1 file** chooser through an opaque ID/content-URI grant, returned to Downloads, exposed no History surface, and left zero exact fixture rows or reverse mappings on `emulator-5554` |
| Shared bookmark-import packaged smoke | Android: `npm run test:smoke:android -- --flow bookmark-import-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --skip-build --flow bookmark-import-smoke.yaml` | Uses one disposable Netscape HTML fixture with platform-native MediaStore/Files seeding and picker navigation to prove preview normalization, explicit selection, 2/1/0 imported/duplicate/failure accounting, public-HTTP upgrade, loopback preservation, saved-row filtering, duplicate omission, no History, and exact fixture cleanup on both simulators |
| Shared Mesh packaged smoke | Android: `npm run test:smoke:android -- --flow mesh-app-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow mesh-app-smoke.yaml` | The same scenario proves explicit enable/disable, offline identity and profile creation, immediate process restart, vault recovery, canonical `ms1.` suggestion/open handling, and no History. It passes in both complete simulator suites; focused Android execution also rejects Capacitor method-payload and Mesh credential-method logs |
| Shared Summer App notification packaged smoke | Android: `npm run test:smoke:android -- --flow summer-app-notification-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow summer-app-notification-smoke.yaml` | Enables the Guide app's browser-owned notification switch, opens the shared TypeScript page through its structured widget, submits the bounded request, receives the native `shown` result, and proves no History surface in both complete simulator suites |
| Shared website notification packaged smoke | Android: `npm run test:smoke:android -- --flow website-notification-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow website-notification-smoke.yaml` | The same top-level loopback fixture and Maestro scenario request the browser-owned exact-origin decision, receive Allow, submit a bounded native delivery request, retain the decision across restart, expose notification revocation in Settings, and prove no History. It passes on Android and on the iPhone 17 Pro / iOS 26.5 simulator |
| Shared protected app-setting smoke | Android: `npm run test:smoke:android -- --flow summer-app-secret-smoke.yaml`; iOS: `npm run test:smoke:ios:remote -- --flow summer-app-secret-smoke.yaml` | Saves a disposable Guide secret, proves sanitized page state, force-stop/relaunch recovery, reset, and no History on both simulator hosts; Android additionally checks device logs for zero literal-secret or Capacitor `methodData` exposure |
| Android signed install/upgrade rehearsal | `npm run test:release:android:emulator -- --avd <Summer_Release_name> --base-build-number <code>` | Refuses physical/ordinary devices, uses a temporary signer, and proves consecutive signed release APK clean-install plus same-key in-place upgrade and persisted tab/bookmark/app setting on a dedicated AVD; this is not Play evidence |
| Android tablet packaged smoke | `npm run test:smoke:android:tablet` | Selects only an emulator, preserves any existing display override, applies tablet dimensions, runs the shared Library flow, and restores the prior display state |
| Local iOS packaged smoke | `npm run test:smoke:ios` | Maestro workflow against an available local iOS simulator |
| Remote iOS packaged smoke | `npm run test:smoke:ios:remote` or append `-- --flow <name.yaml>` for one validated focused flow | Builds web assets, transfers only the committed iOS project, tracked Maestro flows, fixtures, and pinned runner dependencies, runs `xcodebuild`, installs on the configured Mac simulator, and executes the standard plus custom notification, release-upgrade, and process-loss recovery phases |
| Remote signed iOS device preparation | `npm run test:device:ios:remote -- --device <UDID> --team <team-ID>` | Requires an explicit physical device/team, builds and verifies an `iphoneos` signature, and installs/launches through Apple `devicectl`; manual evidence remains required and provisioning mutation is opt-in |

A completed native slice also needs its focused Java/Kotlin or Swift contract
checks when added. The current root Android commands do not by themselves name a
standalone Gradle verification task, so record the exact Gradle/Android Studio
proof used until a repository command is added.

### Required scenarios

- cold start, navigation, background, restore, and process recreation;
- schema migration and corrupt records;
- permission allow, deny, revoke, and OS-versus-site denial;
- download/file cancellation and failure;
- deep-link cold, warm, duplicate, and invalid delivery;
- app eligibility, supported roles, blocked capabilities, and malformed output;
- website-to-Capacitor/native bridge isolation; and
- one host-neutral sample app on desktop, Android, and iOS.

### Release states

Track these separately:

1. TypeScript and native builds pass;
2. deterministic and packaged tests pass;
3. application is signed;
4. store package validation passes;
5. upload completes;
6. store review/certification completes;
7. staged production rollout begins; and
8. install and upgrade are verified from the published store artifact.

A passed isolated-AVD rehearsal proves local signed clean-install and same-key
upgrade behavior only. It does not satisfy store validation, upload, review,
rollout, or published-artifact upgrade evidence.
A successful local build or upload is not evidence of production installation.


## Related documents

- [Desktop-to-mobile strategy](desktop-to-mobile-strategy.md)
- [Summer Mobile implementation and security boundary](../packages/summer-android/README.md)
- [Summer Browser architecture](summer-architecture.drawio)
- [Browser profiles](browser-profiles.md)
- [Summer App development](summer-app-development.md)
- [Summer App submission and platform compatibility](summer-app-submission.md)
- [Chrome extension compatibility](chrome-extensions.md)
