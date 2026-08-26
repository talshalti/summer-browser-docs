# Chrome WebExtensions: desktop and phone differences

> **Current Version 57 boundary:** desktop still uses Electron/Chromium's
> extension engine. Mobile package management remains dynamic and generic, but
> execution requires an engine-owned WebExtension runtime with an enforceable
> isolated-world CSP. Android WebView and iOS 15–18.3 fail closed and display
> `unsupported-engine`; iOS 18.4+ `WKWebExtension` integration is in progress,
> while Android execution is blocked by the current no-emulation/no-alternate-
> engine constraint. The detailed V46–V56 rows below are historical compatibility
> implementation records, not currently enabled execution. See
> [Mobile WebExtension engine migration](mobile-web-extension-engine-migration.md).

Status: Summer 2.1.2 mobile compatibility Version 56

Summer uses the same bundled **Extensions Manager** Summer App on desktop,
Android, and iPhone. The package, routes, review language, installed-extension
records, and manager controls are shared. The execution engine behind that app
is different on phones, so sharing the manager does not mean that every Chrome
extension behaves identically on every platform.

The central product rule is:

- installation is dynamic and general-purpose on every platform;
- compatibility depends on the APIs and manifest behavior implemented by the
  current host;
- no mobile extension is enabled through an extension-specific allowlist or a
  hard-coded Dark Reader/uBlock path.

## At a glance

| Area | Desktop Summer | Android Summer | iPhone Summer |
| --- | --- | --- | --- |
| Host engine | Electron/Chromium extension loader | Android WebView plus Summer's compatibility runtime | WKWebView/WebKit plus Summer's compatibility runtime |
| Manager UI | Shared Extensions Manager Summer App | Same app and stored review model | Same app and stored review model |
| Chrome Web Store | Dynamic signed CRX download | Dynamic signed CRX3 download | Dynamic signed CRX3 download |
| Local CRX | Available | Hidden | Hidden |
| Load unpacked | Available | Hidden | Hidden |
| Per-extension code | None required | None required | None required |
| Package resources | Loaded by Chromium | Exact CRX stored by SHA-256 and indexed natively | Exact CRX stored by SHA-256 and indexed natively |
| Website-visible package resources | Chromium enforces manifest `web_accessible_resources` | Version 48 checks the exact packaged path and authenticated HTTP(S) initiator against MV3 `resources` plus `matches`, or the declared `extension_ids` for another extension; `use_dynamic_url` uses a runtime-scoped opaque host | Same Version 48 policy, independently enforced by Swift/WebKit |
| Privileged same-package fetch and XHR | Chromium resolves extension-owned URLs directly | Version 49 permits only `GET`/`HEAD` from an authenticated background or extension page to its stable or current dynamic host, with native package lookup, bounded transfer state, and 404 for a missing path | Same Version 49 policy, independently enforced by Swift/WebKit |
| Background/page fetch and XHR | Chromium applies its extension network and CSP model | On Android 14/API 34+, Version 48 brokers bounded cookie-free HTTP(S) requests through cache-disabled `android.net.http.HttpEngine` only from an authenticated background or extension page, requiring an effective host permission for every URL and redirect hop; Android 7-13 fail closed for this broker only | Same Version 48 contract over an ephemeral cookie-disabled `URLSession` |
| Content scripts | Chromium worlds/frames according to supported Electron behavior | Version 46: manifest `all_frames`, `world`, `match_about_blank`, and `match_origin_as_fallback`; isolated declarations use the extension world and `MAIN` declarations use a separate bridge-free page-world source | Same Version 46 manifest subset using distinct `WKContentWorld`/page-world sources |
| Background | Chromium extension background/service-worker behavior available through Electron | Version 56 schema v8 serves a reserved PAGE-world generated document: manifest-ordered external classic paths or one private stable-own module import run under validated CSP; trusted initial-document load or complete module evaluation plus exact native load completion releases the generation FIFO | Same shared v8 path-carrier/readiness policy over an exact WKWebView/controller/object/generation; neither host has idle suspend/wake or killed-app parity |
| `offscreen` | Desktop supports Summer's reviewed Chrome reason set in a sandboxed hidden extension page, including reason-specific audio inactivity | Version 56 exposes only Promise `createDocument`, `closeDocument`, and `hasDocument` to an authenticated current MV3 background or top-level owned page with required `offscreen`; only exact `WORKERS` is accepted. One hidden PAGE document per extension/eight globally gets provisional runtime messaging/Ports and verified package modules/Workers under strict CSP/navigation/network bounds | Same shared V56 API, quotas, restricted runtime, package/CSP policy, and process-only lifecycle over an independently bound WKWebView; final Mac/Xcode and live-iPhone behavior remain pending |
| `runtime.reload()` | Chromium reloads the calling extension and applies its rapid-reload protection | Version 51 exposes the zero-argument, synchronous-`undefined` method to backgrounds, pages, and authenticated content scripts; native replies before an async target-only restart, clears only memory-session state/queued session changes, recreates owned runtime pages, preserves durable target state and ordinary website tabs, and terminates/suppresses the sixth accepted call in ten seconds | Same Version 51 contract, with a monotonic runtime epoch and exact WKWebView/controller/document revalidation; sibling extensions and ordinary website tabs remain live |
| `runtime.Port` | Chromium owns Port routing and process/service-worker lifecycle | Version 52 uses a native-owned same-extension registry with opaque non-authority endpoint IDs, OPEN-to-ACTIVATE ordering, per-port FIFO, local-silent/peer-once disconnect, and exact document/epoch/generation teardown on navigation, reload, or renderer loss | Same Version 52 contract, independently enforced by Swift/WKWebView; neither host supports cross-extension Ports, `connectNative()`, idle-worker wake, or killed-app wake |
| Extension commands | Chromium owns browser-window accelerators and its shortcut-management UI | Version 53 exposes `commands` only when the manifest owns that key; callback/Promise `getAll()` and ready-background `onCommand` use foreground physical keyboard input, `default` shortcuts, Android key normalization, deterministic conflicts, and inactive reserved/global/media/AltGr-shaped shortcuts | Same bounded API over dynamic foreground `UIKeyCommand`; `mac` then `default` resolution maps Command/MacCtrl/Alt to Command/Control/Option, with no OS-global registration and no live-iPhone behavior claim |
| Compiled wrapper migration | Chromium owns its internal generated bootstrap | Version 56 requires compiled-source schema v8; schema-v7/older or unversioned records remain visible but disabled as load-failed until signed-package reinstall/reparse | Same shared persistent record policy; Swift never receives a pre-v8 source as enabled configuration |
| Per-document request authority | Chromium owns document/world identity internally | Android establishes a fresh browser-owned token/secret before package source and binds calls to the exact native world, reply proxy, receiver, committed top document, or background generation | Version 51 uses an exact five-field READY envelope and exact four-field `context.request` outer envelope for every direct runtime/storage call; Swift compares its token with the current background, popup, owned page, or verified top document before decoding the inner request |
| Ordinary child-frame broker | Chromium authenticates eligible content-script frames | Static child injection remains available in the supported Version 46 subset, but Version 51 runtime/storage broker authority is top-frame-only; cross-origin child calls fail closed | Same deliberate top-frame-only ordinary authority while authenticated WebKit child/related-frame lifecycle remains incomplete |
| Fair work admission | Chromium schedules extension work internally | Global ceilings are paired with per-extension admission caps across scarce work classes; DNR and user-script read transfers allow at most two slots per extension within each eight-slot pool | Matching per-extension-below-global policy for streamed reads, runtime/website proofs, language, network/resource, and other bounded work so one package cannot occupy every slot |
| Platform info | Chromium reports its native Chrome platform values | `runtime.getPlatformInfo()` reports `android` plus mapped primary ABI | Reports Chrome-valid `mac` plus actual architecture as a documented best-effort Darwin mapping because Chrome has no `ios` enum value |
| API breadth | Broad Chromium/Electron surface | Explicit Summer subset | Same intended Summer subset, implemented independently in Swift |
| Request rules | Chromium declarative net request engine | Summer's bounded indexed block subset with persistent static-selection/dynamic updates and memory-only session updates | Summer's bounded `WKContentRuleList` block subset with transactional static-selection/dynamic/session updates |
| DNR regex capability probe | Chromium compiles supported `regexFilter` expressions through its RE2-backed DNR engine | Version 54 exposes callback/Promise `isRegexSupported()` only in the privileged permission-gated DNR namespace, but every valid-shaped request returns frozen `{isSupported: false, reason: "memoryLimitExceeded"}` without compilation or native IPC; static regex rules are omitted and mutable additions rejected | Same shared fixed-negative wrapper behavior and zero `condition.regexFilter` capacity; this is not an RE2 or native-regex parity claim |
| Blocking `webRequest` | Depends on desktop Electron support | Not implemented | Not implemented |
| Storage access levels | Chromium keeps persistent per-extension policy for local, sync, and MV3 session; local/sync default untrusted-accessible and session defaults trusted-only | Version 55 exposes `setAccessLevel()` on each manifest-available area, permits mutation only from exact privileged contexts, persists policy across reload/update/disable/restart, checks it for every authenticated top-frame isolated-content operation/event, and uses bounded crash-retryable deletion tombstones so removal clears data/package/permission/policy before same-ID confirmation; one mutation's change envelope is capped at 64 KiB, and a nonempty mutation needing an unready background fails before write if its protected 256-event/1 MiB FIFO cannot reserve capacity | Same Version 55 contract, crash-retryable cleanup, bounds, and atomic backpressure with exact WKWebView/controller/document-token/scope rechecks before operations and asynchronous event calls; page world remains excluded |
| `storage.local` | Chromium extension storage, content-accessible by default | Native extension-scoped persistent storage, content-accessible by default under persistent access policy | Same independently enforced native policy and persistent values |
| `storage.sync` | Desktop implementation may later connect to Summer device sync; content-accessible by default | Chrome-shaped device-local namespace today, content-accessible by default under persistent access policy | Same independently enforced native policy and device-local values |
| `storage.session` | Chromium MV3 behavior, trusted-only by default | MV3 memory-only values; Version 55 allows a privileged context to grant/revoke authenticated isolated-content access without changing value cleanup | Same policy/value separation and exact current-document checks |
| Tabs and window queries | Chromium uses numeric tab IDs and real browser window types | Tab IDs are opaque UUID strings; Version 47 validates `discarded`, `lastFocusedWindow`, and every valid `windowType` against one normal undiscarded window | Same UUID-string tab IDs and one-normal-window filtering |
| Extension windows/devtools | Desktop-shaped extension UI is possible | Version 47 supplies read-only `windows.get/getCurrent/getLastFocused/getAll`, constants, optional `populate`, and filtering; no mutations, events, devtools, or fake popups | Same deliberately read-only single-window model |
| File-scheme status | User-manageable desktop grant when reviewed and enabled | Privileged `extension.isAllowedFileSchemeAccess()` reports `false` | Reports `false` |
| Context menus | Desktop broker supports CRUD/click delivery, but some Chrome semantics remain first-pass | Persistent generic registry; trusted link/image long-press composition; opaque revision-bound clicks | Persistent generic registry; trusted link long-press composition; opaque revision-bound clicks; image/selection remain WebKit-owned |
| Notifications | Basic desktop broker with extension-scoped create/update/clear/events | Basic OS notifications, two actions, persisted scoped registry, immutable private click/dismiss routing | Basic OS notifications, two actions, persisted scoped registry, WebKit-background click/dismiss routing |
| Downloads | Broad Summer-record broker: start/search/pause/resume/cancel/open/show/erase/remove/icon subset and events | Version 43 native subset: start/search/cancel/open/show/terminal-history erase and events; sanitized basename only | Same Version 43 API subset over background `URLSession`/foreground `WKDownload`; sanitized basename only |
| User scripts | Explicitly approved persistent/session registration in isolated worlds; custom messaging/CSP remain limited | Version 44: `register`, `getScripts`, `update`, `unregister`, world configuration, three document phases, dedicated isolated worlds, persistent/session lifetime, and explicit per-extension approval | Same Version 44 API and approval model using distinct `WKContentWorld` instances |
| `activeTab` | Chromium mints temporary tab authority after defined user gestures | Version 45: memory-only extension/tab/origin grant from a trusted action, popup, or validated applicable context-menu click; Version 53 also mints it after an accepted physical command activation. The grant covers tab metadata and supported scripting host authority on that origin only | Same authority model, independently enforced by Swift/WebKit |
| Privileged user gesture | Chromium owns transient activation semantics | Version 50 consumes a native five-second monotonic, one-shot grant for `permissions.request` and download `open`/`show`, bound to the exact extension version and WebView document or background generation; the JS Boolean is schema only | Same policy using system uptime and exact WKWebView/controller/page or background-generation binding; queued event delivery cannot restart expiry |
| Permission popup lifecycle | Chromium owns popup/prompt presentation | The exact popup is suspended while trusted Vue chrome presents the decision, restored only after exact response/source/version/document/foreground checks, or closed by a native 30-second watchdog | Same exact suspension/restoration and 30-second watchdog contract in Swift/WebKit |
| `scripting` | Chromium `executeScript`/CSS behavior | Version 46: bounded inline or verified packaged files, `frameIds`/`allFrames`, `MAIN`/`ISOLATED` execution, AUTHOR CSS, and per-frame native host checks; Android child IDs are world-local | Same Version 46 API subset over WebKit frame identities; no `documentIds` or USER CSS |
| Verification status | Existing desktop extension subsystem | Historical V51-V55 focused contracts/builds/device probes remain recorded below. V56 final native/security gates, exact-APK packaged offscreen fixture, and real-extension offscreen behavior are pending; no V56 Android device pass or final security freeze is claimed | Historical V51-V55 source/build evidence remains recorded below. V56 Swift source review is not an Xcode or runtime result; a fresh Mac/Xcode Simulator gate and live-iPhone behavior are pending and not claimed |

The 24,963,237-byte Version 50 APK was installed on `Summer_API_36` /
`emulator-5554`. Direct Capacitor asset synchronization passed. The broader
`android:sync` npm wrapper failed earlier only because the unrelated Captain
Word package could not resolve its Vitest types; that is not a WebExtensions or
native-sync failure. The Android smokes establish the listed behaviors only,
not whole-extension support.

The Version 48 verification record is preserved as historical evidence: before
the final Android security patches, the shared/aggregate suite passed 391/391
tests across 39 files, `vue-tsc`, and the Vite production build, and the iOS
source-contract suite passed 46/46. Post-freeze Android native-contract, JDK
compile, and `assembleDebug` reruns and the connected-Mac build did not execute
because the execution-service allowance was exhausted; no Version 48 Android or
iPhone device behavior smoke ran. Version 47 remained that milestone's last
completed device gate: 373 tests across 38 files, Android JDK 21 assembly/APK
installation, Android native contracts 49/49, iOS source contracts 44/44, a
signed Dark Reader 4.9.129 Android focused smoke, and a fresh iPhone Simulator
build/install in 110.6 seconds; live iPhone behavior was unverified.

## Installation versus compatibility

On mobile, a valid Chrome Web Store package can reach the review screen even if
it declares APIs that Summer does not implement. The manager reports unsupported
features before confirmation. Installing the package means that its identity,
archive, manifest, and requested access passed Summer's checks; it does not mean
that all of its behavior is supported.

Desktop delegates more of extension loading and execution to Chromium through
Electron. Mobile cannot embed Chrome's extension engine into Android WebView or
iOS WebKit, so Summer parses the same package and supplies a clean-room,
best-effort compatibility runtime.

## Version 56 WORKERS offscreen boundary

Desktop Summer delegates a broad offscreen surface to its Electron/Chromium
host and supplements it with a bounded hidden-document manager. Phone Version
56 deliberately implements a narrower common Android/iPhone contract. A
Manifest V3 package must list `offscreen` in its own required permissions; an
optional declaration does not expose the namespace. Only an authenticated
current background, popup, options page, or top-level same-extension tab gets
the Promise-only `createDocument`, `closeDocument`, and `hasDocument` methods.
Content scripts, websites, `MAIN` world code, stale/child documents, and the
hidden document itself do not. The sole supported reason is the exact
one-element `WORKERS` array; phone Summer does not approximate desktop audio or
the other Chrome reasons.

Both phone hosts admit one creating, active, or closing document per extension
and eight globally. The canonical URL must identify a same-extension packaged
HTML file,
may retain its query and fragment, and may not contain credentials, a port,
ambiguous/traversing path syntax, or another package host. Creation waits at
most 25 seconds for both the initial main-frame load and a native ready
handshake. The runtime is installed before package code so a module can use
top-level `await runtime.sendMessage()` before the creation Promise resolves.
The PAGE document receives only `runtime.id`, `getURL`, `sendMessage`,
`connect`, `onMessage`, `onConnect`, and non-enumerable `lastError`.

Android serves the initial page, external modules, Workers, WASM, and other
verified resources through a responder bound to the exact hidden WebView,
extension, document, generation, and package host. Because Android WebView
treats a nonstandard CSP origin as scheme-only, standalone `'self'` tokens are
translated to the explicit `summer-extension:` scheme source in each policy;
the exact native responder remains the cross-extension boundary. iPhone uses
the matching exact WKWebView/scheme-handler binding. On both, the sanitized
manifest extension-pages CSP is intersected with a stricter Summer package-only
script/Worker floor. Inline/remote scripts, direct connections, frames,
navigation, popups, forms, downloads, and arbitrary base/object destinations
remain blocked.

Direct engine network loading, cookies, caches, and credentials are disabled.
Eligible HTTP(S) `fetch()`/XHR may use only the existing cookie-free privileged
broker after manifest network-CSP and current host-permission checks, including
every redirect; Android's remote broker still requires API 34+. This does not
widen website-visible resources or give the hidden document general-purpose
native bridge authority.

An accepted creation belongs to the extension even if the caller disappears.
It may survive replacement of its background, unrelated configuration work,
and an unchanged exact-compatible owning-extension configuration. Explicit
close or `window.close()`, the owning extension's
reload/update/disable/removal, incompatible reconfiguration, load/navigation
failure, renderer/process loss, and browser exit tear down its exact Ports and
pending work. Unlike desktop service-worker/offscreen behavior, this process-
only view has no guarantee of running or waking while the phone app is
suspended or killed. Schema v8 is required; pre-v8 wrappers remain visible but
disabled until signed-package reinstall/reparse.

V56's final Android packaged fixture/real-extension smoke, final native/security
gate, Mac/Xcode Simulator build, and live-iPhone behavior remain pending. The
implementation/source state alone is not a completed cross-platform parity or
security-freeze claim.

## Version 54 fail-closed DNR regex boundary

Desktop Summer retains its RE2-backed DNR matching and real
`isRegexSupported()` result. Mobile Version 54 instead exposes a
shape-compatible probe only in privileged extension contexts that already own
the permission-gated `declarativeNetRequest` namespace. Its exact
`regexOptions[, callback]` object requires an own string `regex`, allows only
optional Boolean `isCaseSensitive` and `requireCapturing`, and throws
`TypeError` synchronously for malformed calls.

For every valid-shaped request, the Promise resolves or the one-shot
asynchronous callback receives the frozen value
`{isSupported: false, reason: "memoryLimitExceeded"}`. Callback invocation
returns `undefined`. The shared generated wrapper does not compile or inspect
the pattern and sends no native request, so Android and iPhone have identical
constant-work behavior.

This is deliberately a zero-capacity contract. An entire static DNR rule with
`condition.regexFilter` is omitted during package compilation, while a dynamic
or session rule addition containing it is rejected. Returning `true` would
therefore advertise filtering that neither native host enforces. The fixed
negative result gives extensions a safe Chrome-shaped feature probe without
claiming RE2, Java, Swift, or Chrome regex parity.

Generated wrappers advance to compiled-source schema v5. Existing schema-v4
and older records stay visible but disabled as `load-failed` until the signed
package is reinstalled and reparsed; editing stored metadata cannot promote an
older wrapper into the new contract.

V54 passes 508 package tests across 41 files, package typecheck and production
build, 84 focused Android native contracts, and 77 focused iOS source contracts.
The final APK at
`packages/summer-android/android/app/build/outputs/apk/debug/app-debug.apk` is
24,963,236 bytes, was built at `2026-08-13T07:53:55.5619353Z`, has SHA-256
`C089F133A293F6444094F571E19A57A242977286EC1295497942F227D64B0787`, and
was installed on Android 16 `emulator-5554`.

Signed uBlock Origin Lite passed in 149.5 seconds with the former missing
`isRegexSupported()` method error absent, the supported `Network advertisement
blocked` result visible, and the
independent trusted-only `storage.session` warning remaining; its artifact is
`C:\Users\Shalti\.maestro\tests\2026-08-13_105515`. Signed Dark Reader's
lifecycle passed in 261.2 seconds while its known `regexps` error remained; its
artifact is `C:\Users\Shalti\.maestro\tests\2026-08-13_105806`.

The standard `android:sync` pre-step encountered unrelated missing Vitest
typings in `capw`; the final package build, Capacitor copy, and Gradle assembly
were run directly and passed. The V54 remote-Mac build is pending because source
transfer requires explicit approval. This record therefore claims no V54 Mac
or iPhone build and no live-iPhone behavior.

## Version 53 foreground commands boundary

Desktop Summer's Chromium/Electron host owns browser-window accelerators and
its own shortcut-management behavior. Mobile Version 53 instead exposes a
bounded `chrome.commands` / `browser.commands` API only in privileged contexts
when the manifest has its own `commands` key. Both callback and Promise forms
of `getAll()` are supported. Standard `onCommand` events go only to the exact
ready background, carry the current sanitized tab, and originate only from a
foreground native hardware-key event; websites and content scripts cannot
forge this route.

Android resolves `default` suggestions. It accepts exact, non-repeating
hardware keyboard/D-pad key-down input in a resumed focused Activity and
normalizes `Ctrl` or left `Alt`, optionally with `Shift`. Right Alt and
`Ctrl+Alt` are rejected as AltGr-shaped input, and soft, virtual, fallback, and
editor input is not eligible. iPhone resolves `mac` before `default`: default
`Ctrl` maps to Command, while mac `Command`, `MacCtrl`, and `Alt` map to
Command, Control, and Option. Its active visible controller owns dynamic
`UIKeyCommand` objects and uses the HID press lifecycle to prevent repeat
delivery.

Bindings resolve by stable extension ID followed by the parsed manifest's
ECMAScript `Object.entries` order. Browser-reserved shortcuts and conflicts are
reported as unassigned. A valid `global: true` suggestion is also inactive;
mobile never registers an OS-global shortcut. Modifier-free media suggestions
are valid manifest data but inactive, and cannot invoke an action alias. The
version-correct MV3 `_execute_action` or MV2 `_execute_browser_action` uses the
trusted toolbar-action path without emitting `onCommand`. Successful native
physical activation mints exact-tab `activeTab` and five-second user-gesture
authority only after the background event or popup/action path is accepted.

Each extension is limited to 128 retained command records, four non-media
suggested commands, and 64 KiB for both canonical native configuration and the
complete `getAll()` projection. Configuration, reload, disable/removal,
background-generation loss, app/view focus loss, and termination clear or
rebuild bindings, held-key state, and authority. Compiled-source schema v4 is
required; the v3-to-v4 migration preserves current v4 records but disables
stale/unversioned records until signed-package reinstall/reparse.

V53 passes 152/152 shared commands contracts, 83/83 Android native contracts
plus JDK 21 compilation, 76/76 iOS source contracts, and 503/503 integrated
package tests across 41 files plus TypeScript checking. Shared/Android/iOS
security reviews report FREEZE/no P1/P2. A fresh 80.0-second remote-Mac gate
passed production bundle/typecheck, Capacitor iOS copy, Xcode iOS Simulator
build, and simulator installation. The final 24,963,238-byte V53 debug APK,
built at `2026-08-13T07:03:51.2031454Z` with SHA-256
`25963128A571F3B77D263622CE0527C5B3449CFB126B87FDC5534CB6F9DAF575`,
was installed successfully on Android 16 `emulator-5554`. On that exact APK, signed
uBlock Origin Lite passed in 157.7 seconds with its former
`commands.onCommand` TypeError absent; its later independent `storage.session`
access warning remained. Signed Dark Reader's tested
full lifecycle passed in 263.2 seconds without an exception involving the
`commands` namespace or `commands.getAll()`; its existing `regexps` error
remained after reload. The retained artifacts are
`C:\Users\Shalti\.maestro\tests\2026-08-13_100437` and
`C:\Users\Shalti\.maestro\tests\2026-08-13_100750`, respectively. These are
focused results, not whole-extension or physical-shortcut proof. Physical
external-keyboard commands remain unverified on Android and iOS, and no
live-iPhone V53 behavior is claimed.

## Package handling

Desktop Chromium owns its installed extension directory and resource loading.
For a mobile Web Store install, Summer does the following:

1. The native host downloads at most 32 MiB from Google's HTTPS update service.
2. Trusted shared code verifies the CRX3 developer proof and requested extension
   ID, parses the manifest, reports access, and requires confirmation.
3. Android or iPhone stores the exact CRX under an extension-scoped SHA-256
   digest.
4. At native configuration time the host rechecks the digest, CRX3 envelope,
   archive directory, paths, compression, duplicates, encryption, symbolic
   links, entry/expanded limits, and CRCs.
5. The host exposes verified files only on that extension's private
   `summer-extension://` origin. Localized CSS may replace an existing CSS file;
   it cannot add arbitrary native resources.
6. Removing the extension removes its local/sync/session state and native CRX
   directory.

Legacy mobile records without a native digest continue to use the earlier
bounded resource transaction so upgrades do not silently break installed data.

Compiled generated source is a separate trust boundary from the CRX digest.
Version 51 introduced wrapper schema v2, Version 52 used schema v3 for the
native Port contract, Version 53 used schema v4 for commands, and Version 54
used schema v5 for the DNR probe. Version 55 stamps newly parsed source as
schema v6; the superseded initial Version 56 draft used v7, and the current
offscreen/PAGE-background wrapper is schema v8.
A record with a missing or older
source schema remains inspectable but is disabled as
load-failed, and the manager requires reinstall/reparse before it may be enabled
again. A valid stored package is not enough to promote JavaScript generated
under the old document-authority or offscreen contract.

## Version 51 reload, document, and fairness boundary

Desktop Chromium owns `runtime.reload()` and extension-document identity inside
its extension process model. Mobile Version 51 supplies the same public
zero-argument, synchronous-`undefined` call as a narrow compatibility method.
The inner runtime request has exactly `version`, `type`, and `id`. Android and
iPhone validate that source as a current background, popup, owned extension
page, or authenticated content script, acknowledge it, and only then schedule
replacement work. The acknowledgement does not imply that a stale document may
continue using the new runtime.

The normal restart boundary is deliberately smaller than reinstall, disable,
or browser-wide configuration. It advances only the target runtime epoch,
clears target `storage.session` and queued session-change notifications, revokes
ephemeral request/document/gesture/network/scripting endpoints, and recreates
the target background and owned pages. It preserves target local/sync storage,
permission decisions, alarms, menus, enabled static rulesets, dynamic/session
rules, and non-session queued browser events. It neither invokes the global
extension configuration path nor reloads ordinary website tabs, and siblings
continue with their existing documents and native state.

Both mobile hosts count accepted reloads against a monotonic rolling ten-second
window. The first five use the normal path. The sixth is acknowledged and then
terminates the exact installed target, drops its runtime queue and owned pages,
removes its action/bindings, and records a suppression that ordinary events and
renderer recovery cannot clear. The user must explicitly reconfigure/re-enable
the extension or install a replacement identity.

Each mobile document now starts with native-created authority before package
code. The shared raw READY object has exactly five fields:
`version`, `type`, `documentToken`, `timeOrigin`, and `topTimeOrigin`. iPhone's
direct runtime/storage route wraps every non-READY inner operation in an exact
four-field `context.request` object and rejects it unless the outer token is
still the token of the current background generation, popup, owned page, or
verified top document. Android enforces the corresponding proof through an
ordered browser identity bootstrap and exact native-world/proxy/receiver
binding. Navigation, replacement, or reload invalidates document A before
document B is promoted, closing delayed-call ABA paths.

For ordinary websites, the currently provable identity is the top-level
document only. Supported static matching can still place code in eligible child
frames, but cross-origin child runtime/storage broker calls fail closed. That is
an explicit mobile gap rather than authority inferred from an origin string or
transient engine object.

Finally, V51 audits scarce native work as a shared-system resource. Per-extension
limits sit below global limits for pending messages, website proofs/calls,
scripting/language/storage work, network/package transfers, and streamed DNR and
user-script reads. The read transports reserve no more than two of each global
eight slots for one extension, and end/expiry/reconfiguration/source-loss paths
release the same accounting. This makes fairness a runtime invariant rather
than an assumption that installed packages cooperate.

Version 51 completed its focused gate with Android native contracts 73/73, iOS
native source contracts 63/63, shared package/migration contracts 79/79, JDK
compilation, and the connected-Mac iOS build-only gate. Signed Android Dark
Reader passed review/install, visible activation, cold restart persistence,
disable, re-enable, removal, and absence in 217 seconds. Signed uBlock Origin
Lite passed review/install and the tested static network block in 140.6 seconds,
including recovery after its background invoked `runtime.reload()`. This remains
focused Android evidence rather than whole-extension or live-iPhone parity.

## Version 52 native Port boundary

Desktop Chromium owns `runtime.Port` identities and process routing. On mobile,
Version 52 gives every same-extension connection two native-issued opaque
endpoint IDs. Those IDs are lookup selectors, never authority: Android and
iPhone reauthorize each OPEN, ACTIVATE, MESSAGE, and DISCONNECT against the exact
installed version, source document, runtime epoch, native receiver, and
background generation. The opener's success is published only after CONNECT is
queued, while ACTIVATE prevents peer code from running before the opener has its
Port object.

Messages are FIFO within a Port, including an accepted final message before a
later disconnect. A caller's explicit `disconnect()` emits no local
`onDisconnect` and notifies the peer exactly once. Navigation, reload,
background replacement, renderer crash, or another endpoint loss instead emits
one disconnect to the exact surviving current endpoint. A closed Port throws on
`postMessage()`. If teardown carries an error, `runtime.lastError` exists only
during the applicable disconnect listeners and is cleared immediately after.
Duplicate and stale native events are inert.

Each host admits at most 64 Ports per extension and 256 globally. Pending data is
bounded to 64 messages and 256 KiB per Port, 256 messages and 1 MiB per
extension, and 2,048 messages and 8 MiB globally, with a 30-second lifetime.
Cross-extension Ports
and `connectNative()` are absent. A content document from before
`runtime.reload()` cannot be adopted into the new generation; a fresh top-level
navigation must establish new authority. Ordinary website broker calls remain
top-frame-only. Port recovery does not imply Chrome idle service-worker
suspend/wake or killed-app delivery.

V52's final focused gates pass: shared Port contracts 91/91, Android Port
contracts 76/76 plus JDK compilation, iOS Port contracts 69/69, exact
content-scope contracts 115/115 shared, 78/78 Android plus JDK compilation,
70/70 iOS, and 272/272 integrated, plus the 41-file/486-test package gate,
TypeScript check, and package diff check. Security reviews returned FREEZE/no
P1/P2 for shared/Android/iOS Ports, exact content scopes, and the viewport
transition. The fresh remote-Mac gate passed production bundle/typecheck,
Capacitor iOS copy, Xcode iOS Simulator build, and simulator installation in
94.7 seconds.

The 24,963,237-byte final debug APK at
`packages/summer-android/android/app/build/outputs/apk/debug/app-debug.apk` was
built at 2026-08-13 05:01:02 UTC, has SHA-256
`7625131D6B4AC38533E94375619B86C06ED83BFCB9460ED58B3661EDD38914BA`, and was
installed on Android 16 `emulator-5554`. On that exact APK, signed Dark Reader
passed install/content activation/cold restart/disable/re-enable/removal in
246.5 seconds; the deterministic signed Port fixture passed ordered echo, pre-open
FIFO, local disconnect, background-reload replacement, and navigation teardown
in 175.1 seconds; and signed uBlock Origin Lite passed install and the visible
`Network advertisement blocked` assertion in 130.8 seconds. These are focused Android results, not whole-
extension or live-iPhone compatibility claims.

## Historical Version 50 execution and authority boundary

Desktop Chromium owns its internal extension-world bootstrap. At the Version 50
milestone, Summer's shared shim captured indirect global `eval`, the bound
native post function, and the browser scheduler before package code executes.
It pre-serializes the exact `runtime.backgroundReady` envelope while the token is
still private. Classic package source then runs as a global classic script in a
separate lexical environment from the shim closure. Replacing JSON, Promise,
the channel method, scheduler, bind, or shared prototypes cannot expose the
token or suppress the browser-owned one-shot readiness notification. Version 56
schema v8 supersedes this package-execution path with a CSP-governed PAGE
document and exact classic/module path carriers.

Callback-form APIs also differ internally from earlier mobile versions: the
Promise bridge now attaches separate success and failure handlers. A success
callback that throws is logged once and never re-entered with a fabricated
`runtime.lastError`.

For gesture-gated APIs, the JavaScript `userGesture` Boolean is compatibility
shape only. Android uses a monotonic native clock and iPhone uses system uptime
to enforce the same five-second, single-use grant. Each grant is bound to the
enabled extension version and exact page document or background generation;
the original absolute expiry follows a queued action/context event. Permission
requests and download presentation consume that native grant. Navigation,
source replacement, reconfiguration, timeout, or first use revokes it.

The optional-permission UI also has an exact native lifecycle. If the request
originated in a popup, Summer suspends that same popup before showing trusted
Vue chrome, then restores it only after the response and source/version/document
remain current in a foreground app. Failure or stale state closes the exact
privileged view and cancels its work. A native 30-second watchdog guarantees
cleanup if UI completion never arrives.

## Version 49 background lifecycle boundary

Desktop Chromium/Electron owns extension service-worker/event-page scheduling.
Mobile instead keeps one hidden event-document host per enabled extension and
now supervises that host as an exact native generation. The generated bootstrap
receives a fresh, non-persisted 32-character hexadecimal capability at each
native configuration.
Native code accepts one matching `runtime.backgroundReady` only after classic
top-level execution returns or a module import succeeds. An early transport-ready
message may bind the authenticated native receiver, but cannot release events or
runtime calls.

After authenticated readiness, Android and iPhone drain one canonical FIFO to
the exact extension version, view/controller, receiver, and generation. The queue
is bounded to 32 extension queues, 256 events per queue, 64 KiB per event, and
1 MiB per queue. Browser events may evict only droppable browser events; lifecycle
events and accepted action/context-menu work are protected. Background storage
mutations use an exact `storage.changed` runtime envelope, while live content and
popup contexts retain their established storage-change route. Readiness-waiting
one-shot runtime sends and initial port-open requests keep the 64 KiB payload,
128-pending, and 30-second bounds; they fail on generation loss and are not
replayed into a replacement.

If Android reports a dead WebView renderer or iOS reports terminated WebKit
content, Summer destroys that hidden host and creates a fresh one. Queued
browser/storage events remain eligible for the replacement; stale evaluations,
acknowledgements, receivers, and callbacks cannot dequeue or answer work.
Recovery allows at most three recovery attempts in 60 seconds and schedules one
retry when that
window expires. This improves process-loss recovery while the browser is alive;
  it is not Chrome idle service-worker suspend/wake scheduling and does not wake a
  killed app. At the Version 49 milestone, persistent `runtime.Port` objects did
  not receive native endpoint-loss disconnect when their background generation
  disappeared; Version 52 supersedes that historical limitation while the app is
  alive.

Version 47 gives extension backgrounds a safe package base for relative
`fetch()` and `XMLHttpRequest` URLs, including paths relative to a nested
background entry. Version 48 adds the bounded website-resource policy and
host-permission network broker described below. These are generic manifest and
runtime capabilities; they do not identify or special-case any extension.

## Version 49 privileged package-fetch boundary

An authenticated background or extension page may use `fetch()` or XHR to read
its own verified packaged resources. The shared runtime accepts only
`summer-extension:` URLs on that extension's stable ID host or its current
runtime-dynamic host, and both Android and iPhone independently revalidate the
privileged source, enabled extension, current background generation when
applicable, exact host, and canonical packaged path. Cross-extension reads are
rejected even when the target resource is web-accessible.

The local broker supports `GET` and `HEAD` only, caps one resource at 8 MiB,
and keeps at most 8 transfers/8 MiB retained per extension and 16 transfers/
16 MiB globally for 30 seconds. A valid missing path returns a normal 404
response rather than a transport error. Query strings may reach the resource
URL but do not change the canonical archive path; credentials, ports, fragments,
request bodies, and traversal are rejected. This is package access, not the
Version 48 remote HTTP(S) broker, so it works without Android 14. Ordinary
websites and content scripts receive no privileged route and retain their
existing `web_accessible_resources` and page-CORS boundaries.

## Version 48 package-resource boundary

An extension's own authenticated background and extension pages may read exact
resources from their verified package. Every other caller needs an explicit
manifest declaration:

- a Manifest V3 `web_accessible_resources` rule supplies one or more safe
  `resources` globs and at least one allowed caller set: HTTP(S) origin patterns
  in `matches`, Chrome extension IDs in `extension_ids`, or both;
- the Manifest V2 string list is treated as its legacy public shape: the listed
  resources are available to matching HTTP and HTTPS websites and extensions;
- website and content-script loads must be `GET` or `HEAD`, must name an exact
  packaged resource, and must carry trustworthy initiator evidence. Missing,
  malformed, opaque, or conflicting origin/referrer evidence fails closed;
- another extension must be authenticated as that exact extension and match
  the declaration's `extension_ids`. Knowing a package URL is not authority;
- an allowed website response carries only that website's origin in
  `Access-Control-Allow-Origin`, varies on `Origin`, disables caching, and uses
  `nosniff`. Denials are indistinguishable 404 responses rather than package-
  existence disclosures.

The stable package URL is
`summer-extension://<extension-id>/<path>`. When a path matches a rule with
`use_dynamic_url: true`, `runtime.getURL(path)` instead uses
`summer-extension://<32-hex-id>.summer-extension.dynamic/<path>`. The opaque ID
is unique, memory-only runtime state and can rotate after a browser restart,
extension reload, disable/re-enable, or replacement. Callers must obtain the
current URL instead of storing or constructing it. A dynamic-only declaration
does not authorize the stable URL; a separate non-dynamic rule can independently
authorize the same path. The opaque hostname reduces durable URL correlation,
but it is not a secret or a substitute for the resource/caller checks.

Policy input is bounded to 128 rules, 256 values in each rule field, 4,096 total
resource/caller references, and safe resource globs of at most 1,024 characters.
Unsupported schemes and path-shaped website matches are rejected at inspection;
website `matches` are normalized HTTP(S) origin patterns ending in `/*`.

## Version 48 host-permission network boundary

Only an authenticated extension background, popup, options page, or extension-
owned tab receives the native HTTP(S) `fetch()`/XHR broker. A content script's
request remains an ordinary WebView/WKWebView request governed by the page's
origin and CORS policy, and an ordinary website has no broker message route.
Each initial URL and every followed redirect must match a current required host
permission or a declared optional host permission the user actually granted.
Disable, removal, replacement, or permission revocation invalidates work already
in progress. Redirect following is capped at five hops; `redirect: "error"`
rejects the first redirect, `manual` is unavailable, an undeclared or non-HTTP(S)
target rejects, and cross-origin redirects remove `Authorization`.
Success reconstructs a normal `Response` with the final URL, redirected flag,
status/status text, filtered headers, and buffered body; the XHR subset maps the
same result into its supported response modes.

Brokered requests are intentionally cookie-free. Cookie request headers and URL
credentials are rejected, `credentials: "include"` and XHR
`withCredentials` are unavailable, native cookie stores are disabled, and
`Set-Cookie` / `Set-Cookie2` never reach extension code or browser storage. The
Android broker is available on Android 14/API 34+ and uses a cache-disabled
`android.net.http.HttpEngine`, which ignores ambient `Authenticator`,
`CookieHandler`, and default TLS globals. Android 7-13 fail closed for this
broker only. Extension pages on both mobile hosts also block remote subframes
and form submissions. The current CSP integration is deliberately bounded. An
absent CSP, or a valid CSP with neither `connect-src` nor fallback `default-src`,
leaves the broker eligible. When either network directive applies, its ASCII-
tokenized source list must contain `*`; host-only lists, duplicate/malformed
directives, commas/newlines, and Unicode or other non-ASCII whitespace fail
closed. Native effective-host authorization still applies on every hop. This
is not full Chrome network, navigation, or CSP parity.

The current cross-platform request limits are an 8,192-byte URL, 64 headers and
32 KiB of request-header text, a 1 MiB request body, a 25-second timeout, and 16
retained transfers per extension/host. Responses are capped at 128 headers and
64 KiB of response-header text plus up to 8 MiB of decoded body; 8 MiB plus
one byte rejects. Methods are limited to `GET`, `HEAD`, `POST`, `PUT`, `DELETE`,
and `OPTIONS`. Fetch `no-cors`, `same-origin`, integrity, and manual-redirect
modes reject; XHR is asynchronous-only and supports text, JSON, array-buffer,
and blob responses with timeout/abort events.

## Implemented mobile API families

Version 46 also expands manifest execution itself: `content_scripts` honors
`all_frames`, `world`, `match_about_blank`, and `match_origin_as_fallback` for
supported HTTP(S) and related-frame cases. Static `MAIN` declarations are kept
out of the extension API runtime and compiled into a separate bounded page-world
source with no `chrome`, `browser`, Summer, Capacitor, or native bridge.

Version 47 adds package-relative background resource loading and truthful
single-window discovery. It improves generic feature detection and background
bootstrap compatibility without pretending that a phone has desktop popup or
devtools windows.

The current Android and iPhone compatibility runtime supplies bounded portions
of these Chrome-shaped APIs:

- `runtime` messaging, same-extension ports, `getURL`, options-page opening,
  callback/Promise platform information, and validated `onInstalled` /
  `onStartup` background bootstrap, plus a validated `setUninstallURL()` no-op
  that never retains or opens an uninstall survey;
- `storage.local`, device-local `storage.sync`, privileged memory-only
  `storage.session`, and change events;
- current-window `tabs` queries and core lifecycle operations/events with
  permission-filtered URL/title metadata and opaque UUID-string IDs; Version 47
  validates `discarded`, `lastFocusedWindow`, and every valid `windowType` and
  filters them against the one normal undiscarded mobile window;
- read-only `windows.get`, `getCurrent`, `getLastFocused`, and `getAll` with
  `WINDOW_ID_NONE`, `WINDOW_ID_CURRENT`, optional bounded `populate`, and valid
  `windowTypes` filtering over that one real window; mutation methods, events,
  and fabricated popup windows are absent;
- privileged `extension.isAllowedFileSchemeAccess()` callback/Promise forms
  reporting `false`;
- top-frame `webNavigation` queries and core navigation events;
- inline-function or verified packaged-file `scripting.executeScript` in
  `MAIN`/`ISOLATED`, and inline or packaged-file `insertCSS` / `removeCSS`, with
  default-main-frame, explicit `frameIds`, or `allFrames` targeting after native
  per-frame authorization;
- transient `activeTab` authority for tab URL/title and supported
  scripting after a validated browser-chrome gesture or accepted foreground
  physical command activation; `activeTab` is a
  permission and does not create a JavaScript namespace;
- manifest-dependent privileged `commands.getAll()` callback/Promise forms and
  exact-ready-background `commands.onCommand` from foreground native hardware
  input, with deterministic inactive conflicts/reserved shortcuts and bounded
  action-alias handling;
- action/browser-action clicks, popups, title, badge, enabled state, and packaged
  path icons, including safe background-entry-relative icon resolution;
- background-relative `fetch()` and XHR access to verified same-extension
  package resources, plus bounded cookie-free HTTP(S) requests from authenticated
  backgrounds/pages when every target has a current effective host permission;
- website and cross-extension package-resource access controlled by exact
  Manifest V2/V3 `web_accessible_resources` resource/caller declarations, with
  runtime-scoped opaque URLs for `use_dynamic_url` resources;
- persistent alarms with a mobile minimum cadence;
- package-owned `i18n` messages and bounded local language detection;
- browser-owned optional permission prompts for implemented API and host
  families;
- persistent permission-gated `contextMenus` / `menus` CRUD and click events,
  with trusted phone long-press composition for links on both platforms and
  images on Android;
- permission-gated basic `notifications` create/update/clear/getAll and
  permission-level queries, with click, close, and up-to-two-button events;
- permission-gated `downloads` start/search/cancel/open/show/erase with
  created/changed/erased events; `open` also requires `downloads.open`, file
  presentation requires a live extension-page user gesture, and results expose
  only a sanitized basename rather than a local path;
- explicitly approved, permission- and host-gated `userScripts` registration,
  query, update, unregister, and world-configuration methods. Persistent
  registrations survive browser restart, session registrations do not, and
  package-version changes clear registrations before new extension code runs.
  Version 45 stages batch mutations atomically, applies an aggregate registry
  bound, and rebuilds future-document bindings after permission changes without
  forcibly reloading ordinary tabs;
- a block-only subset of `declarativeNetRequest`, including persistent
  `getEnabledRulesets` and `updateEnabledRulesets` for declared rulesets,
  persistent `getDynamicRules` / `updateDynamicRules`, and memory-only
  `getSessionRules` / `updateSessionRules`. Mutable updates use bounded chunked
  transactions because the authenticated per-message limit remains 64 KiB.
  Version 54 also supplies a fixed-negative callback/Promise
  `isRegexSupported()` probe without adding regex-rule enforcement.

This list describes API shapes, not perfect Chrome semantics. The authoritative
limits and detailed behavior are in
[Mobile WebExtension compatibility — Beta](mobile-web-extensions.md).

Mutable rule lifecycle now follows the important Chrome boundaries: dynamic
rules survive browser restarts and extension upgrades; session rules are
memory-only and reset on browser exit or extension update; enabled static
ruleset choices survive a browser restart but reset to manifest defaults when
the extension version changes. Android rebuilds its native request index after
an accepted transaction. iPhone compiles the complete candidate WebKit list
before it persists or swaps any live state.

## Important mobile gaps

The largest differences from desktop Chrome-style behavior are:

- no general Chrome extension service-worker suspend/resume lifecycle;
- no cookie-bearing or unbounded extension network client, no content-script
  CORS bypass, and no complete Chrome CSP integration; an absent CSP or a valid
  CSP without an applicable `connect-src`/`default-src` leaves the broker
  eligible, while an applicable directive must contain `*` and malformed or
  host-only policy fails closed;
- Android 7-13 have no privileged remote HTTP(S) broker, and extension pages on
  both mobile hosts currently block remote subframes and form submissions;
- no phone presentation yet for page, selection, editable, media, action, or
  iPhone image contexts, although compatible items can be stored in the generic
  registry;
- no `documentIds` target selection or `USER`-origin CSS through
  `chrome.scripting`; Android child-frame IDs are local to each execution world
  and cannot be correlated between `ISOLATED` and `MAIN`;
- static related-frame matching for `about:blank`, `about:srcdoc`, `data:`,
  `blob:`, and `filesystem:` is best-effort and fails closed when WebView/WebKit
  cannot expose a trustworthy parent, opener, or referrer;
- `userScripts` supports arbitrary registered code in `USER_SCRIPT` worlds,
  including optional child-frame injection, but not Chrome's `MAIN` world,
  custom world CSP, custom messaging, `execute()`, or broader host-pattern
  coverage than the extension currently owns;
- no blocking `webRequest`, request-event stream, or arbitrary native proxy;
- no DNR redirect, allow-priority, header mutation, feedback, regex filters, or
  mutable actions/conditions beyond the documented safe block subset; Version
  54's `isRegexSupported()` method truthfully reports the zero regex capacity;
- no image/list/progress notification templates and no automatic permission
  prompt from extension code;
- no extension window creation, update, removal, or events, no fake popup
  windows, and no side panels, devtools, bookmarks, history, cookies,
  identity, native messaging, VPN, or filesystem API surface; the implemented
  `windows` namespace is read-only and exposes only Summer's one normal window;
- the mobile downloads API has no cookie inheritance, absolute path, nested
  filename, `saveAs`, custom method/body/headers, pause/resume, icon, danger
  acceptance, file deletion, `onDeterminingFilename`, or full query support;
  `show` presents the file through trusted mobile OS UI rather than revealing a
  desktop folder, and history erase currently retains active transfers;
- no automatic cross-device `storage.sync` transport yet;
- no promise that a Chrome Web Store package using unsupported APIs will work
  merely because it can be inspected and installed;
- iOS distribution remains subject to Apple review rules for downloaded code,
  independent of technical behavior.

Android and iPhone also have host-specific differences. Android requires a
WebView provider with isolated execution-world support and fails closed when it
is absent. Its child-frame numbers are assigned independently within the
isolated and main-world routes; only frame 0 is stable across those worlds. Its
remote extension broker separately requires Android 14/API 34+.
iPhone uses WebKit content worlds and compiles the supported request
rules into `WKContentRuleList`; its complete signed uBlock smoke must still be
rerun once the connected Mac's independent XCTest/Maestro launch baseline is
healthy.

## What parity means for this project

“Mobile parity” is not satisfied by drawing the desktop manager on a phone. It
means progressively implementing extension capabilities behind the same app and
same reviewed package model, with these gates for each capability:

1. generic manifest/API behavior rather than an extension-name check;
2. independent native authorization and bounds on Android and iPhone;
3. regression tests for malformed and hostile package/input cases;
4. packaged emulator behavior evidence on both platforms;
5. an explicit compatibility warning until the complete behavior is proven.

The ranked real-extension targets—from simple theme extensions through Dark
Reader, Stylus, SponsorBlock, uBlock Origin Lite, password managers, and wallet
extensions—are maintained in
[the mobile compatibility target table](mobile-web-extensions.md#compatibility-targets-easiest-to-hardest).
The Version 48 website-resource and permission-gated network slice moves several
targets forward, but it is not a
claim that Dark Reader, uBlock Origin Lite, or any other real package is fully
compatible.

See also [desktop Chrome extension support](chrome-extensions.md) for desktop
installation and Electron-specific behavior.
