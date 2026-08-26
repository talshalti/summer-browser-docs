# Chrome extension compatibility

> **Mobile Version 57 security boundary:** the shared manager can still
> download, verify, review, persist, update, disable, and remove arbitrary signed
> CRX3 packages. Mobile does not execute them unless the native capability
> handshake proves an engine-owned WebExtension runtime, isolated-world CSP,
> and Chrome CRX3 support. Android WebView and iOS 15–18.3 report
> `unsupported-engine`; iOS 18.4+ `WKWebExtension` integration is in progress.
> Android execution is blocked under the current no-emulation/no-alternate-
> engine constraint. Mobile V46–V56 runtime sections below are
> historical implementation records, not current execution support. See
> [Mobile WebExtension engine migration](mobile-web-extension-engine-migration.md).

Summer can install signed CRX packages, download Chrome Web Store packages, or
load unpacked Chrome extensions through the bundled **Extensions Manager**
Summer App. This compatibility layer is built on Electron's supported
extension APIs; it does not provide full Chrome compatibility.

The same `sap://extensions-manager/` Summer App package also runs in Summer's
Android and iOS hosts. Mobile has a narrower best-effort runtime, but its Web
Store installation is dynamic: any signed CRX3 can be inspected and installed
without an allowlist. Version 46 manifest content scripts honor `all_frames`,
`world`, `match_about_blank`, and `match_origin_as_fallback` in the supported
HTTP(S)/related-frame subset. `ISOLATED` declarations run in extension-specific
native worlds; static `MAIN` declarations are compiled into a separate minimal
page-world source with no `chrome`, `browser`, Summer, Capacitor, or native
bridge. Classic background scripts run in a bounded per-extension event
document, and content-to-background `runtime.sendMessage` / `onMessage` plus
ordered same-extension `runtime.connect` ports, native-backed `storage.local`,
and device-local `storage.sync` are available.
Version 47 resolves background-relative `fetch()` and `XMLHttpRequest` package
paths from the verified background entry directory. Version 48 additionally
enforces Manifest V2/V3 `web_accessible_resources` for authenticated website and
cross-extension callers, including runtime-scoped opaque hosts for
`use_dynamic_url`, and brokers bounded cookie-free HTTP(S) fetch/XHR only for an
authenticated background or extension page whose current effective host
permissions cover every target and redirect.
Version 49 adds a generation-scoped native supervisor for that hidden background
host. Each configuration receives a fresh 32-character hexadecimal capability
embedded only in the fixed generated notifier and native descriptor. Native code
accepts exactly one matching
`runtime.backgroundReady` after the classic top-level source has returned or the
module import has succeeded, then releases queued work only to the exact
extension version, native view/controller, authenticated receiver, and
generation. Browser
and background-only storage events share a bounded FIFO (at most 32 extension
queues, 256 events per queue, 64 KiB per event, and 1 MiB per queue); lifecycle
events
and already accepted user actions are protected from ordinary-event eviction.
One-shot runtime sends and initial port-open requests may wait for readiness
within their existing 64 KiB, 128-pending, and 30-second bounds, but fail rather
than replay when that generation is lost. Android and iPhone replace a
terminated hidden renderer with
a fresh host and retry at most three times in 60 seconds, retaining queued
browser/storage events for the replacement. This remains a persistent hidden
event-document compatibility layer, not Chromium idle service-worker suspend/
wake or killed-app delivery. Version 52's native Port registry, described below,
owns endpoint-loss disconnect while the app is alive.
Version 50 historically hardened classic background execution around captured
indirect `eval`; Version 56 supersedes that bootstrap with schema v8. Native now
serves a reserved CSP-governed PAGE document containing manifest-ordered
external classic paths, or the shim privately imports one stable-own module and
waits for complete evaluation. A captured trusted initial-document load or
private import completion plus exact native load completion gates readiness;
rejected module startup sends an authenticated failure and is torn down.
Chrome-shaped callbacks use distinct Promise success and
failure handlers. If a success callback itself throws, Summer logs it exactly
once instead of invoking the callback again under a fabricated
`runtime.lastError`.
Version 50 also moves gesture authority fully into each native host. A
JavaScript `userGesture` field remains a Boolean-shaped compatibility hint where
the schema expects it, but either value grants no authority.
`permissions.request()` and download `open()`/`show()` consume a native,
five-second monotonic, one-shot grant bound to the extension version and exact
page document or background generation. Grants originate only in trusted
action, context-menu, native page-touch handling, or an accepted Version 53
physical command activation; queued background events
retain the original absolute expiry instead of restarting it when delivered.
Navigation, source replacement, reconfiguration, expiry, or first use revokes
the grant. When a popup requests optional access, Summer hides that exact popup
while trusted browser chrome presents the decision, restores it only after the
exact response/source/version/document remains current, and otherwise closes
it. A native 30-second watchdog tears down a stranded suspended popup and its
runtime/network work, preventing an invisible privileged page.
Version 51 adds a general-purpose mobile `chrome.runtime.reload()` and
`browser.runtime.reload()` implementation. It is exposed in the same background,
extension-page, and authenticated content-script contexts as Chrome, accepts no
arguments, returns `undefined` synchronously, and emits an exact three-field
inner request. Android and iPhone authenticate the source, acknowledge that
request before restart work, and asynchronously replace only the calling
extension's runtime. A normal reload clears its memory-only `storage.session`
values and queued session-change notifications, revokes ephemeral document and
runtime endpoints, and recreates its hidden background and extension-owned
pages. It does not reload ordinary website tabs, reconfigure sibling
extensions, clear durable local/sync storage, revoke accepted permission
decisions, or discard alarms, menus, and static/dynamic/session request rules.
The sixth accepted reload inside a rolling ten-second window instead terminates
and suppresses the exact installed target until an explicit manager
configuration/re-enable, matching Chromium's bounded rapid-reload behavior and
preventing event or renderer recovery from reviving a reload loop.

The same version advances the generated mobile wrapper to compiled-source
schema v2. Records generated by an older or missing schema remain visible but
are disabled with a load-failed state; enable/reload is rejected until the
signed package is reinstalled and parsed into v2. This is an intentional
fail-closed migration, not an automatic trust upgrade. Before package code,
each host now establishes fresh per-document authority. READY registration uses
one exact five-field envelope. On iPhone every direct runtime or storage request,
including privileged background, popup, and owned-page calls, is wrapped in an
exact four-field `context.request` envelope and must carry the token currently
bound to that native document/generation. Android supplies the equivalent
current-document proof through its native world and receiver binding. Stale
documents and delayed replies cannot act on a replacement generation.

Ordinary website broker authority is currently limited to the authenticated
top-level document. Static Version 46 matching may still inject an eligible
child script, but a cross-origin child cannot obtain runtime/storage broker
authority and fails closed. Shared scarce-work classes also carry
per-extension admission caps below their global ceilings, including streamed
DNR and user-script reads, so one dynamically installed package cannot reserve
all slots needed by another.

Version 52 advanced the compiled wrapper to schema v3 and replaced the earlier
message-route Port approximation with a native-owned same-extension endpoint
registry. Each connection receives two opaque native endpoint IDs; they are
selectors, not capabilities, and every request is reauthorized against the exact
extension version, source document, runtime epoch, and background generation.
OPEN publishes success only after the peer CONNECT is queued, then ACTIVATE
allows delivery, so peer code cannot race ahead of the opening response. Messages
remain FIFO per port. Local `disconnect()` produces no local event and notifies
the peer exactly once; navigation, reload, background replacement, renderer
crash, or other endpoint loss notifies the exact surviving endpoint once. A
closed Port rejects later sends, and `runtime.lastError` is scoped only to an
error-bearing `onDisconnect` listener invocation.

Native admission is bounded to 64 Ports per extension and 256 globally. Queued
messages are limited to 64 messages and 256 KiB per Port, 256 messages and 1 MiB
per extension, and 2,048 messages and 8 MiB globally, with a 30-second lifetime.
On that mobile runtime, cross-extension connections and `connectNative()` are
not implemented. A content
context from before `runtime.reload()` cannot attach to the replacement runtime;
it needs a fresh top-level navigation. Ordinary website authority remains
top-frame-only, and this work does not add idle service-worker or killed-app wake.
At that milestone, stored pre-v3 wrappers remained visible but failed closed
until signed-package reinstall/reparse. Version 56's current schema-v8 boundary
is described below; changing a stored schema number never upgrades trust.

Version 56 adds a bounded mobile-only Manifest V3 `chrome.offscreen` /
`browser.offscreen` compatibility slice. The package must own the required
`offscreen` permission. Only an authenticated current background or top-level
extension-owned page receives Promise-only
`createDocument({url, reasons: ["WORKERS"], justification})`,
`closeDocument()`, and `hasDocument()`; optional-only declarations, callbacks,
content/page worlds, the hidden document itself, and every other Chrome reason
remain unavailable. Summer admits one creating, active, or closing document per
extension and eight globally and waits at most 25 seconds for the exact initial
main frame plus provisional runtime.

The hidden PAGE document receives only `runtime.id`, `getURL`, `sendMessage`,
`connect`, `onMessage`, `onConnect`, and non-enumerable `lastError`. That runtime
is installed before package code, so an external module can use top-level-await
messaging before `createDocument()` resolves and can create a verified packaged
Worker. The canonical initial URL must select a same-extension packaged HTML
resource without credentials, a port, path ambiguity/traversal, or another
package host. A dedicated native responder is bound to the exact view,
extension object, document, generation, and host for its modules, Workers, WASM,
and other package files.

Mobile enforces the sanitized manifest extension-pages CSP together with a
stricter Summer package-only script/Worker floor. Direct engine networking,
frames, later navigation, popups, forms, downloads, and arbitrary base/object
destinations are blocked. Eligible HTTP(S) fetch/XHR uses only the existing
cookie-free privileged broker after current manifest-CSP, host-permission, and
per-redirect checks; Android still requires API 34+ for that remote broker.

An accepted creation belongs to the extension even if the caller disappears.
It may survive background replacement, unrelated configuration changes, and an
unchanged exact-compatible owning-extension configuration, but explicit
close/`window.close()`, owning-extension
reload/update/disable/removal, incompatible reconfiguration, load/navigation
failure, renderer/process loss, and browser exit tear down the exact Ports and
pending work. This is process-only phone state, not idle-worker, suspended-app,
killed-app, audio, or general background-execution parity. Pre-v8 wrappers stay
visible but disabled until signed-package reinstall/reparse. Final V56 Android
packaged/real-extension, native/security, Mac/Xcode Simulator, and live-iPhone
gates remain pending; this is not a completed V56 device or security-freeze
claim.

Version 49 also routes privileged same-extension package `fetch()` and XHR
through a native-validated local-resource transaction. Only `GET` and `HEAD`
may address the caller's stable extension host or its current runtime-dynamic
host. Each resource is capped at 8 MiB; native state is capped at 8 in-flight
transfers and 8 MiB retained per extension, 16 transfers and 16 MiB globally,
with a 30-second lifetime. A valid but missing package path completes as 404.
This route neither requires Android 14 nor widens `web_accessible_resources`:
websites and content scripts remain isolated from the privileged broker.
Storage changes propagate among the same extension's live isolated contexts.
Every isolated mobile context also receives bounded package-owned
`chrome.i18n` / `browser.i18n` message substitution with Chrome's exact
preferred-locale, parent-language, and declared-default fallback order,
recursive manifest and manifest content-script CSS localization, `escapeLt`,
live Summer UI/accepted-language reporting, and Chrome's extension-ID and bidi
special messages. Malformed or ambiguous alternate catalogs fail inspection;
separately served packaged CSS uses the same locale fallback during native
synchronization and refreshes after a Summer language change. Every isolated
context can also call bounded callback or Promise `i18n.detectLanguage`; Summer
uses Android 10+ `TextClassifier` or Apple's local `NaturalLanguage` recognizer,
returns at most three validated candidates, and returns an explicit unreliable
empty result on Android 7 through 9. This is a best-effort Chrome-shaped result,
not identical CLD confidence semantics, and it does not add a web service.
Backgrounds also receive bounded `tabs.get`, `query`, `detectLanguage`,
`create`, `update`, `reload`, `remove`, and `sendMessage` operations for
current-window lifecycle and same-extension content messaging. `detectLanguage`
uses a fixed browser-owned reader capped at 4,096 DOM text nodes and 8,192
UTF-16 code units, revalidates the exact tab lifecycle before and after native
detection, and returns only a normalized primary language or `und`; raw page
text is not exposed to the extension. Android and iOS return tab URL/title only
when the extension requires `tabs`, has a matching effective host permission,
or currently holds a browser-minted `activeTab` grant for that exact tab and
origin; optional declarations and content-script matches do not silently grant
access.
Mobile tab IDs remain opaque UUID strings. Version 47 validates the
`discarded`, `lastFocusedWindow`, and complete Chrome-valid `windowType` query
fields, then filters them against Summer's one normal undiscarded mobile window.
Privileged backgrounds and extension pages also receive read-only
`windows.get`, `getCurrent`, `getLastFocused`, and `getAll`, including
`WINDOW_ID_NONE`, `WINDOW_ID_CURRENT`, optional bounded `populate`, and valid
`windowTypes` filtering. Summer does not invent popup windows or expose window
mutations/events.
Backgrounds also receive native `tabs.onCreated`, `onUpdated`, `onActivated`,
and `onRemoved` events with the same URL/title permission filtering. Favicon
metadata is not implemented. Extensions that require `scripting` can execute a
bounded inline function or verified packaged JavaScript files in `ISOLATED` or
`MAIN`, and insert/remove bounded inline or packaged CSS, against the default
main frame, explicit `frameIds`, or `allFrames`. Android/iOS independently
authorize every target frame against effective host access or a same-origin
`activeTab` grant; the grant never substitutes for the separate `scripting`
permission. `documentIds` targeting and `USER` CSS remain unsupported, and
Android child-frame IDs are local to each execution world rather than
correlated between `ISOLATED` and `MAIN`. Manifest-declared `action` /
`browser_action` entries appear in trusted mobile browser chrome and either
deliver permission-filtered `onClicked` tab data to the isolated background or
open a verified manifest `default_popup` in a temporary, same-extension native
view. Popup pages receive the existing scoped runtime, storage, tabs, and
scripting compatibility APIs; their navigation and resources remain on the
installed extension's `summer-extension://` origin and use a strict default
CSP. Declared `options_ui.page` / `options_page` documents open through the same
temporary host from either the shared manager or `runtime.openOptionsPage()`.
Action and `browserAction` code can also change default/per-tab title, popup,
enabled state, badge text, and badge background color, and trusted phone chrome
updates from the native authoritative state. Verified manifest icons and
global/per-tab `setIcon({path})` changes are served from bounded packaged image
resources. Version 47 safely resolves relative background calls from the
background entry's package directory; `ImageData` icons, remaining action
methods, and arbitrary extension
window behavior remain unsupported. Privileged extension contexts can now use
`tabs.create` or `tabs.update` to display a verified same-extension packaged
HTML document in an ordinary mobile browser tab. Android and iOS bind that tab
to the caller's extension ID, install only its isolated APIs, reject cross-
extension or non-HTML main documents, and replace the view with a clean website
view before following HTTP(S). Cold startup restores the exact owned packaged
page through a separate trusted native operation after reviewed extensions are
synchronized; both hosts revalidate enabled ownership and the HTML resource,
and stale records become isolated blank tabs without affecting siblings.
Verified package files are available through
extension-owned `summer-extension://` URLs returned by `runtime.getURL()`, with
exact-path native handlers on Android and iOS. Website/content-script loads must
match a declared resource glob and HTTP(S) initiator; another extension must
match `extension_ids`. A `use_dynamic_url` path receives a memory-only
`summer-extension://<32-hex-id>.summer-extension.dynamic/` host, and its
dynamic-only declaration does not authorize the stable extension-ID URL.
Missing or conflicting initiator evidence and undeclared resources fail closed
as native 404 responses. Unsupported APIs and manifest features are reported as
warnings. Enabled
static DNR rulesets also contribute
a separately chunked native block subset for simple path filters,
`||domain^` exact/subdomain rules, domain-anchored paths, and `requestDomains`,
bounded to 150,000 rules across enabled
extensions. Privileged extension pages can query and persistently change which
declared static rulesets are enabled with `getEnabledRulesets()` and
`updateEnabledRulesets()`; Android rebuilds its index and iOS transactionally
compiles and swaps the combined content rule list. Static block rules also preserve
`excludedRequestDomains` and exact included/excluded `image`, `stylesheet`,
`script`, `font`, `media`, and `xmlhttprequest` types; unidentifiable Android
request types fail open. Other resource types, initiator conditions, broader
DNR syntax, allow/redirect/header actions, and large real-blocker behavior remain
unsupported. Version 39 also exposes `getDynamicRules`, `updateDynamicRules`,
`getSessionRules`, and `updateSessionRules` for that same safe block subset.
Dynamic rules persist across restarts and extension upgrades; session rules are
memory-only and extension-version-scoped. Both hosts use bounded, expiring,
chunked transactions and native revalidation. iOS compiles the complete
candidate WebKit rule list before replacing live state.
Version 40 also exposes callback-or-Promise `runtime.getPlatformInfo()` and
delivers validated `runtime.onInstalled` / `runtime.onStartup` events to the
matching mobile background only after its classic source or module import has
finished bootstrapping. Android reports its Chrome `android` platform and mapped
ABI. Because Chrome's `PlatformOs` has no iOS member, iPhone uses a documented
best-effort `mac` Darwin mapping with the real architecture. Locale refresh,
reload, enable/disable, rollback, and same-version review do not fabricate
lifecycle events.
Version 41 adds a generic permission-gated `contextMenus` / `menus` subset on
mobile: persistent create/update/remove/removeAll state, nested normal,
checkbox, radio, and separator items, per-item and global click listeners, and
trusted browser-owned long-press composition. Android presents compatible link
and image targets; iPhone presents links and preserves WebKit's native image and
selection menus. Each click uses a short-lived single-use opaque token bound to
the current tab revision and waits for the matching background's explicit
post-bootstrap ready signal.
Version 42 adds generic permission-gated basic OS notifications on both phone
hosts: create/replace/update/clear/enumerate/permission queries, at most two
actions, and exact-extension click/close/button delivery. Summer owns the OS
permission prompt and visually attributes each notification to its extension;
extension code cannot trigger the OS prompt silently. It also hardens persisted
context-menu graphs against cycles and accepts the official `tab` declaration
for future trusted tab-menu presentation.
Version 43 adds a generic permission-gated mobile `chrome.downloads` /
`browser.downloads` subset on both hosts. Privileged extension contexts can
start credential-free native HTTP(S) GET downloads, search and cancel the
browser-owned registry, present completed files after an extension-page user
gesture, erase terminal history without deleting files, and receive bounded
`onCreated`, `onChanged`, and `onErased` events after background bootstrap.
`open` also requires `downloads.open`. Unlike desktop Chrome, mobile returns a
sanitized basename instead of an absolute filesystem path, does not inherit
browser cookies for extension-started requests, and does not yet support
`saveAs`, nested paths, custom methods/headers/bodies, pause/resume, file icons,
danger acceptance, file deletion, `onDeterminingFilename`, or the full query
language. `show` uses trusted OS file presentation rather than a desktop folder
reveal, and active transfers remain browser-owned when history is erased.
Version 44 enables the generic, explicitly approved `chrome.userScripts` /
`browser.userScripts` subset on both phone hosts. Any dynamically installed
extension declaring `userScripts` can register, query, update, and unregister
persistent or session code after the user enables **Allow User Scripts** in the
shared manager. Each registration is checked again by native code for extension
identity/version, effective permission, exact owned host patterns, quotas, and
source context, then runs at document start/end/idle in a dedicated Android or
iPhone isolated world. Optional child frames are supported. Package updates
clear old registrations. `MAIN`, custom world CSP/messaging, and `execute()`
remain intentionally unavailable in this stage.
Version 45 adds Chrome-shaped `activeTab` authority without inventing a
`chrome.activeTab` namespace. A trusted Summer action click, action-popup open,
validated extension context-menu click, or accepted Version 53 physical command
activation can mint a memory-only grant for the
exact active ordinary website tab and canonical HTTP(S) origin. That grant
reveals only that tab's URL/title and can satisfy the native host-authority check
for supported scripting on that origin. Same-origin navigation and tab
switching retain it; cross-origin navigation, tab replacement or close, extension
reconfiguration, permission revocation, and process teardown clear it. Version
45 also makes user-script batch commits atomic, bounds the aggregate registry,
rechecks current host authority when injecting, and refreshes future-document
bindings after permission changes without reloading open pages.
Version 46 adds the generic packaged-file, frame-target, and execution-world
slice described above. Static related-frame matching for `about:blank`,
`about:srcdoc`, `data:`, `blob:`, and `filesystem:` is best-effort and fails
closed when WebView/WebKit cannot expose a trustworthy parent, opener, or
referrer. Final Version 46 shared-build, Android emulator, and iPhone simulator
results passed through 364 deterministic mobile tests, the TypeScript/Vite
build, both native compilers, Android APK install/launch, and connected-Mac
iPhone simulator build/install. A signed Dark Reader package installed on
Android but its visible activation remained pending, so this evidence does not
establish Dark Reader or uBlock Origin Lite compatibility.
Version 47 adds the generic background package-base, tab/window-query,
file-scheme-status, uninstall-URL, and relative action-icon behavior described
above. Privileged `extension.isAllowedFileSchemeAccess()` reports `false`, and
`runtime.setUninstallURL()` validates an empty or bounded HTTP(S) URL but does
not retain or open a survey when the extension is removed. Final Version 47
verification passes all 373 deterministic mobile tests across 38 files,
including Android native contracts 49/49 and iOS native source contracts 44/44,
plus
`vue-tsc`, the Vite production build, Android JDK 21 `assembleDebug`, and
emulator APK installation. A signed Dark Reader 4.9.129 Android smoke rendered a
visibly dark fixture with no filtered extension errors. That is a focused
Android result, not complete Dark Reader or broad extension compatibility. A
clean connected-Mac retry built and installed fresh Version 47 source on the
iPhone 17 Pro Simulator in 110.6 seconds; the earlier SwiftPM clone timeout was
transient. The live iPhone Dark Reader Maestro/XCTest driver still hung while the
simulator remained on its home screen, so live iPhone extension behavior remains
unverified.
Version 48's remote broker remains outside content scripts and websites, so
those contexts retain normal page-origin CORS behavior. Broker requests do not
send or store browser cookies, reject credentialed requests, filter Set-Cookie,
and reauthorize each of at most five redirect hops. On Android 14/API 34+ the
privileged broker uses a cache-disabled `android.net.http.HttpEngine`, which
ignores ambient `Authenticator`, `CookieHandler`, and default TLS globals.
Android 7-13 fail closed for this broker only; the other supported extension
capabilities remain available. Mobile extension pages also currently block
remote subframes and form submissions as a security compatibility gap. The
current request bounds
are an 8,192-byte URL, 64 headers/32 KiB, a 1 MiB body, 25 seconds, and 16
retained transfers; responses are limited to 128 headers/64 KiB and up to 8
MiB of body. Successful fetches preserve final URL, redirect status, HTTP status,
filtered headers, and buffered body. With no declared CSP, or with a valid CSP
that declares neither `connect-src` nor fallback `default-src`, the broker
remains eligible. When either network directive applies, its ASCII-tokenized
source list must contain `*`; host-only lists, duplicate/malformed directives,
commas/newlines, and Unicode or other non-ASCII whitespace fail closed. Native
effective-host authorization still applies on every hop.
Before the final Android security
patches, the Version 48 shared/aggregate suite passed 391/391 tests across 39
files, `vue-tsc`, and the Vite production build; the iOS source-contract suite
passed 46/46. Those results do not validate the post-patch Android native source.
Post-freeze Android native-contract, JDK compile, and `assembleDebug` reruns could
not execute because the execution-service allowance was exhausted and sandboxed
Gradle could not read its required user configuration. The connected-Mac Version
48 build was also blocked by the execution allowance, and no Version 48 Android
or iPhone device behavior smoke ran. The deterministic local-only fixture and
prepared, non-default Maestro flow are documented in
[`web-extension-network.README.md`](../packages/summer-android/test-fixtures/web-extension-network.README.md).
Version 51 completed its focused verification with Android native contracts
73/73, iOS native source contracts 63/63, and shared package/migration contracts
79/79, followed by the bounded Android navigation-attempt cleanup. The JDK
compile and connected-Mac iOS build-only gate passed. The signed Dark Reader
Android flow passed review/install, visible activation, cold restart persistence,
disable, re-enable, removal, and absence in 217 seconds. The signed uBlock Origin
Lite flow passed review/install and the tested static network-ad block in 140.6
seconds, including recovery from its background's `runtime.reload()` call. These
are focused Android results, not claims of complete extension compatibility;
live iPhone extension behavior remains unverified.

V52's final focused gates pass: shared Port contracts 91/91, Android Port
contracts 76/76 plus JDK compilation, and iOS Port contracts 69/69. Exact
content-script scopes pass 115/115 shared, 78/78 Android plus JDK compilation,
70/70 iOS, and 272/272 integrated. The final package gate passes 486 tests across
41 files, TypeScript checking, and the package diff check. Security reviews
returned FREEZE/no P1/P2 for shared/Android/iOS Ports, exact content scopes, and
the viewport transition. A fresh remote-Mac gate passed the production
bundle/typecheck, Capacitor iOS copy, Xcode iOS Simulator build, and simulator
install in 94.7 seconds.

The final debug APK at
`packages/summer-android/android/app/build/outputs/apk/debug/app-debug.apk` is
24,963,237 bytes, was built at 2026-08-13 05:01:02 UTC, has SHA-256
`7625131D6B4AC38533E94375619B86C06ED83BFCB9460ED58B3661EDD38914BA`, and was
installed on Android 16 `emulator-5554`. On that exact APK, signed Dark Reader
passed install, active content behavior, cold restart, disable, re-enable, and
removal in 246.5 seconds; the deterministic signed Port fixture passed ordered echo,
pre-open FIFO, local disconnect, background-reload replacement, and navigation
teardown in 175.1 seconds; and signed uBlock Origin Lite passed install and the
visible `Network advertisement blocked` assertion in 130.8 seconds. These are focused Android behavior
probes, not whole-extension or live-iPhone compatibility claims.

Version 49's 418/418 and Version 50's 429/429 gates remain preserved historical
evidence. The Android renderer-recovery smoke still requires explicit approval
to expose the localhost WebView CDP socket.

Version 53 adds mobile foreground hardware-keyboard commands without changing
desktop Electron's existing `commands` implementation. On Android and iPhone,
the privileged namespace exists only when the manifest owns a `commands` key.
It supports callback/Promise `getAll()` and exact-ready-background
`onCommand`; websites and content scripts cannot dispatch it. Android resolves
`default` and rejects soft/virtual/repeated and AltGr-shaped input. iPhone
resolves `mac` before `default`, maps Command/MacCtrl/Alt to
Command/Control/Option, and owns its shortcuts through the active visible
`UIKeyCommand` responder.

Mobile conflicts resolve by stable extension ID then parsed manifest order.
Reserved browser shortcuts, media shortcuts, conflicting losers, and valid
`global: true` suggestions remain inactive, with no OS-global registration.
The version-correct MV3/MV2 action alias uses Summer's trusted action path and
does not emit `onCommand`. A successful standard command or action alias mints
exact-tab `activeTab` plus the five-second native user-gesture grant only after
event delivery or popup/action opening succeeds.

Limits are 128 commands, four non-media suggested commands, and 64 KiB for the
canonical native configuration and complete `getAll()` projection. Generated
wrappers advance to schema v4; stale v3/unversioned records remain disabled
until reinstall/reparse, and lifecycle loss clears or rebuilds native bindings.

V53 passes 152/152 shared commands contracts, 83/83 Android native contracts
plus JDK 21 compilation, 76/76 iOS source contracts, 503/503 integrated package
tests across 41 files, and package TypeScript checking. Security reviews report
FREEZE/no P1/P2. A fresh 80.0-second remote-Mac production bundle/typecheck,
Capacitor iOS copy, Xcode Simulator build, and simulator install passed. The
final 24,963,238-byte V53 debug APK, built at
`2026-08-13T07:03:51.2031454Z` with SHA-256
`25963128A571F3B77D263622CE0527C5B3449CFB126B87FDC5534CB6F9DAF575`,
was installed successfully on Android 16 `emulator-5554`. On that exact APK, signed
uBlock Origin Lite passed in 157.7 seconds and its former
`commands.onCommand` TypeError was absent; its later independent
`storage.session` access warning remained. Signed Dark Reader's tested
full lifecycle passed in 263.2 seconds without an exception involving the
`commands` namespace or `commands.getAll()`; its existing `regexps` error
remained after reload. Physical external-keyboard command smoke remains
unverified on both platforms, and no
live-iPhone V53 behavior is claimed. See
[Mobile WebExtension compatibility — Beta](mobile-web-extensions.md)
for the exact key normalization, authority, quota, and lifecycle contract.

Version 54 adds a shape-compatible, fail-closed mobile
`declarativeNetRequest.isRegexSupported()` probe without changing desktop
Electron's full DNR implementation. The method exists only in privileged
extension contexts that already receive Summer Mobile's permission-gated DNR
namespace. Its exact `regexOptions[, callback]` contract requires an own string
`regex`, permits only optional Boolean `isCaseSensitive` and
`requireCapturing`, and throws `TypeError` synchronously for malformed calls.
For every valid-shaped call, the Promise form resolves and the callback form
runs once asynchronously with the frozen result
`{isSupported: false, reason: "memoryLimitExceeded"}`; the callback form itself
returns `undefined`.

This probe neither compiles the supplied pattern nor calls either native host.
Mobile's capacity for DNR `condition.regexFilter` remains zero: package
compilation ignores an entire static rule containing `regexFilter`, while
dynamic or session rule additions containing it are rejected. The fixed
negative result is therefore honest feature detection, not a claim of RE2,
Android, iOS, or Chrome regex parity. Generated wrappers advance to schema v5;
schema-v4 and older records stay visible but disabled as `load-failed` until
their signed package is reinstalled and reparsed.

V54 passes the complete 41-file/508-test mobile package suite, package
typecheck and production build, 84 focused Android native contracts, and 77
focused iOS source contracts. The final APK at
`packages/summer-android/android/app/build/outputs/apk/debug/app-debug.apk` is
24,963,236 bytes, was built at `2026-08-13T07:53:55.5619353Z`, has SHA-256
`C089F133A293F6444094F571E19A57A242977286EC1295497942F227D64B0787`, and
was installed on Android 16 `emulator-5554`.

On that APK, signed uBlock Origin Lite passed in 149.5 seconds with the former
missing `isRegexSupported()` method error absent, the visible `Network advertisement blocked`
assertion intact, and only its trusted-only `storage.session` warning remaining;
the retained artifact is
`C:\Users\Shalti\.maestro\tests\2026-08-13_105515`. Signed Dark Reader's tested
lifecycle passed in 261.2 seconds while its known `regexps` error remained; its
artifact is `C:\Users\Shalti\.maestro\tests\2026-08-13_105806`. The normal
`android:sync` pre-step stopped on unrelated missing Vitest typings in `capw`,
so the passing final path used the package production build, Capacitor copy, and
Gradle assembly directly. A remote-Mac V54 build still requires explicit
transfer approval and is pending; no V54 Mac/iPhone build result is claimed.

Version 55 brings the mobile storage access-level class to local, sync, and
MV3 session without changing desktop Chromium ownership. Newly parsed mobile
wrappers use schema v6 and expose `setAccessLevel()` on each area. Local and
sync default to `TRUSTED_AND_UNTRUSTED_CONTEXTS`; session defaults to
`TRUSTED_CONTEXTS`. Only an authenticated privileged extension context may
change the native per-extension policy. Existing authenticated top-frame
isolated content contexts gain or lose operations and change events live, with
native document-token, runtime-epoch, declarative-scope, and policy checks at
each operation and delivery. Page-world websites receive no bridge. Policy
survives reload, update, disable/re-enable, configuration rebuild, and browser
restart, but removal deletes it; session values remain memory-only and keep
their existing cleanup lifecycle. Removal is crash-retryable: a durable bounded
tombstone excludes the ID before native package/data/permission/policy cleanup,
startup repeats that idempotent cleanup, and install confirmation stays blocked
until all tombstones clear. Policy changes emit no storage event. Mobile
requires one mutation's complete serialized change set to fit 64 KiB; larger
`set`, `remove`, or `clear` requests fail atomically rather than truncating or
silently losing their change event, so large removals should be batched. A
nonempty mutation that must notify an unready background also fails before its
write when that extension's protected 256-event/1 MiB FIFO is full; it can be
retried after the queue drains, while no-op mutations still succeed.
V55 passed 540/540 deterministic tests, production build, Capacitor copy, and
JDK 21 Android assembly. The 24,963,237-byte APK has SHA-256
`3C8876AB1BA010F558EE27DA84D8568654EB918FF410F0916C3E79161F9C425C`.
Its deterministic signed storage fixture passed the visible grant/revoke and
exactly-once event lifecycle in 163.5 seconds at
`C:\Users\Shalti\.maestro\tests\2026-08-13_130518`. Signed uBlock Origin Lite
passed in 170 seconds with its visible block assertion and no prior
trusted-only session warning in the bounded log scan; signed Dark Reader passed
its 321.1-second install/restart/disable/re-enable/remove lifecycle. iOS source
contracts pass 85/85, while the V55 Mac/Xcode build and live-iPhone behavior
remain unverified pending explicit source-transfer approval.

Local CRX and unpacked installation remain hidden on mobile, and most browser
APIs are not implemented there yet. See
[Mobile WebExtension compatibility — Beta](mobile-web-extensions.md)
for the exact mobile boundary and ranked compatibility targets, and
[Chrome WebExtensions: desktop and phone differences](desktop-mobile-web-extension-differences.md)
for the side-by-side product contract.

## Use the manager

1. Open **Settings > Apps and widgets > Apps**.
2. Open **Extensions Manager**, or navigate directly to
   `sap://extensions-manager/`.
3. Paste a Chrome Web Store detail URL or 32-letter extension ID, choose a
   local `.crx` file, or choose **Load unpacked** and select a directory
   containing `manifest.json`.
4. Review the extension's requested API access, site access, content-script
   matches, optional access, and compatibility diagnostics before confirming.

## Install recommendations

Desktop Summer can offer a browser-owned install notification after the active
regular tab remains for six seconds on either:

- any exact Chrome Web Store extension detail URL on
  `chromewebstore.google.com` or the legacy `chrome.google.com` host; or
- a curated HTTPS website with a publicly documented Chrome extension.

The initial curated relationships live in
`shared/extensionInstallRecommendations.ts`: Grammarly, 1Password, Bitwarden,
Notion Web Clipper, Evernote Web Clipper, and Todoist. Each record owns the
display name, exact 32-letter Web Store ID, and official domains; websites do
not supply notification text or extension identity. Maintain the list against
the vendors' own pages and matching Web Store listings:
[Grammarly](https://www.grammarly.com/browser/chrome),
[1Password](https://1password.com/downloads/browser-extension),
[Bitwarden](https://bitwarden.com/download/google-chrome-password-manager/),
[Notion](https://www.notion.com/help/web-clipper),
[Evernote](https://evernote.com/web-clipper/web-clipper-for-chrome), and
[Todoist](https://www.todoist.com/help/articles/use-the-todoist-extension-on-your-web-browser-EZERGsoH).

Recommendations fail closed for private or background tabs, an unavailable
manager, already managed extensions (including disabled ones), and every shared
proactive-tip suppression condition. Navigating away cancels the delayed
candidate. A visible prompt consumes the shared proactive education budget;
closing it or choosing **Not now** snoozes extension recommendations for 30
days, while **Don't suggest extensions** disables this category in the local
profile. The policy stores only the category outcome and timestamps, never the
visited URL, site name, or extension ID.

**Review & install** mints a two-minute, single-use, in-memory token and opens
the trusted manager at `sap://extensions-manager/?recommendation=<token>`. The
manager immediately removes the query from browser history, consumes that
browser-owned authority once, downloads the signed CRX through the existing Web
Store path, and opens the normal access review. A website, typed URL, expired
token, or replay cannot start that download. The manager never confirms the
review or enables the extension without the user's second explicit action.

Extension publishers can request this identical prompt from a visible user
action on a top-level HTTPS page with
`window.summi.loadAPI("extensions").suggestExtension(extensionId)`. The browser
accepts only the extension ID, revalidates the current regular foreground tab,
and applies the same installed-extension suppression, proactive-tip budget,
snooze, opt-out, signed-package review, and second-confirmation rules. See the
[`window.summi` reference](window-summi-api.md#browser-provided-extensions-api)
for the complete return and failure contract.

Extension names and descriptions that use Chrome's `__MSG_*__` manifest syntax
are resolved from `_locales/<locale>/messages.json`. Summer follows Chrome's
preferred locale, base language, and `default_locale` fallback order before
showing the review or installed-extension card.

The manager can disable, enable, reload, review, grant or revoke reviewed file-URL access, open a declared options page,
or remove an extension. Removing an extension removes Summer's record, unloads
it, and cleans Summer's browser-owned snapshot when one exists. It never deletes
the selected source directory.

On desktop, an active extension also has a one-shot **Debug popup** action.
Select it, then open that extension's real toolbar popup in the tab where the
problem occurs. Summer keeps the popup alive while detached DevTools open for
both the popup document and its service worker. The debugging request is held
only in process memory and does not grant extension access or persist across a
browser restart. Hosts without Electron DevTools do not expose the action.

Reload also rebuilds only that extension's service-worker registration before
loading its reviewed package again. Toolbar popups perform the same bounded
worker-readiness check and one scoped repair when a registration retained by an
older Summer build cannot start. This preserves the extension's local storage,
site data, settings, and the rest of the browser profile.

Compatibility diagnostics are derived from the manifest before extension code
runs. Summer allows Electron-supported extension APIs such as `runtime`,
`storage.local`, `tabs`, `scripting`, `webRequest`, `webRequestBlocking`,
`management`, and DevTools extension pages. Summer also adds browser-owned
compatibility shims for extension pages and Manifest V3 service workers through
dedicated `chrome-extension://` frame and service-worker preloads. Summer does
not rewrite an extension's manifest, worker, HTML, or other package files.
Each preload exposes a narrowly scoped broker through `contextBridge`, then
installs the compatibility surface in the extension main world before its own
scripts execute.
Managed extension pages use the authenticated main-process broker for
browser-backed APIs and runtime messaging. In an ordinary extension context
where that complete relay is unavailable, Summer preserves a complete native
`runtime.sendMessage`/`onMessage` pair. If neither exists, it uses an
origin-scoped, structured-clone-only `BroadcastChannel` fallback with bounded
payloads, pending calls, and a 30-second response limit. That channel is used
only for same-extension runtime messaging; it never carries privileged worker
API calls or events.
Runtime messages have a 16 MiB per-message limit so wallets can carry bounded
Snap initialization payloads. The browser also enforces a 64 MiB aggregate
pending-message quota and a 30-second reply lifetime to prevent concurrent
large messages from becoming unbounded memory pressure.

The service-worker preload instead exposes one fixed, call-only `contextBridge`
capability. Summer authenticates its `ServiceWorkerMain` sender and bounds each
request and response before the main-process broker handles it. Validated
main-to-worker events use one preload-owned callback captured during worker
bootstrap, before extension code can scuttle ambient globals. The callback is
released with worker teardown. No Electron, Node, filesystem, or raw IPC object
crosses into extension code. Worker permission and command reads remain derived
from the reviewed manifest.
Those shims expose brokered real-profile cookies, bookmarks, browsing history,
download records, alarms, notifications, debugger targets, popup windows,
proxy settings, privacy/content-setting preferences, commands, context menus,
side panels, font settings, permissions, supported browsing-data removal, and
web navigation. The machine-readable registry in
`shared/chromeExtensionCompatibility.ts` is authoritative for each member.
Required Chrome API permissions are warnings rather than install blockers so
real extensions can load and reveal the next compatibility gap at runtime.
Malformed manifests and changed permission fingerprints are still rejected or
sent back through review before code runs.

An enabled extension is loaded into Summer's persistent browsing session on
each launch. Before a Summer-initiated startup, enable, or reload, a changed
permission declaration leaves the extension disabled until it is reviewed. A
broken extension is reported in the manager without preventing the other
extensions or the browser from starting.

After a successful load, enable, or reload, Summer reloads each open matching
HTTP(S) tab once so newly registered content scripts apply without a manual
refresh. Summer Apps, internal pages, DevTools, and non-persistent/private tabs
are excluded, and one tab's reload failure does not fail the extension action.

Enabled extensions that declare a Manifest V3 `action` or Manifest V2
`browser_action` appear beside the address bar. Summer uses the declared title
and nearest 32-pixel icon, keeps additional actions in a keyboard-accessible
overflow menu, and opens a declared popup in the extension's own persistent
session. Popup content has no Node.js or general Summer browser bridge; it
receives only the scoped extension API compatibility preload. It cannot open
new windows or navigate its top-level surface outside its own
`chrome-extension://` origin. Same-extension bootstrap redirects are accepted,
optional content sizing cannot fail popup loading, and internal focus changes
do not close hardened popups; outside pointer-down, Escape, tab changes, unload,
or replacement still close them.

Summer provides browser-owned `chrome.action` and `chrome.browserAction`
state, including titles, icons, popups, badges, enabled state, and per-tab
overrides. Extension pages and Manifest V3 workers mutate that state through
the authenticated broker, and the toolbar snapshot updates from the same
authoritative store. Click-only actions without a declared popup wake their
managed worker when necessary and dispatch `onClicked` with the active Summer
tab.

Right-clicking an extension action opens its matching extension-supplied
`contextMenus` entries followed by browser-owned controls. **Options** appears
only when the active managed extension declares an options page; **Manage
extension** opens and focuses its exact Extensions Manager record; and **Remove
extension…** opens that record's existing confirmation dialog. Summer
revalidates the active tab, action, and managed runtime mapping before showing
the menu and again before opening a browser-owned destination.

Enabled extensions that declare `options_ui.page` or `options_page` expose an
**Options** button in the manager. The browser process resolves the reviewed
relative manifest path against the active Electron extension ID and opens the
resulting `chrome-extension://` page in a normal Summer tab. The manager never
receives filesystem paths or extension runtime IDs.

Extensions with required `alarms` permission can call a limited `chrome.alarms`
implementation from extension pages and Manifest V3 workers. The supported
methods are `create`, `get`, `getAll`, `clear`, and `clearAll`; the supported
event is `onAlarm`. Alarm state persists in browser-owned storage with bounded
per-extension quotas and is reconciled with the managed extension lifecycle.
A firing alarm wakes its managed worker before event delivery when the worker
is not already running.

Extensions with required `declarativeNetRequest` permission can use Summer's
browser-owned static ruleset engine. Summer loads enabled
`declarative_net_request.rule_resources` from the reviewed extension snapshot
and enforces the supported block, allow, redirect, upgrade, and header-rule
subset for HTTP(S), WS, and WSS requests in the same session as the extension.
Static, dynamic, and session rules use one browser-owned decision pipeline;
Summer does not depend on Electron's native static-rule enforcement.
Normalization is fail closed: a rule with an invalid, oversized, or unsupported
condition is skipped instead of being widened into an unconditional action.

Production matching prepares each request once and uses a conservative,
precompiled index keyed by action phase, resource type, and proven hostname
anchors. Ambiguous filters remain in fallback candidate buckets, and the shared
rule matcher is still the final authority, so indexing cannot widen or change a
rule. Ruleset compilation and request matching then run on dedicated Node worker
threads instead of the Electron main event loop. Static rules and the smaller
mutable dynamic/session set compile into separate matchers whose results are
combined deterministically. The main process still validates and normalizes
rules before sending a serializable copy to a worker. The worker client keeps
the current generation active while a standby worker compiles its replacement,
then swaps generations atomically; ordinary mutable updates rebuild only the
mutable matcher. Updates have a 120-second deadline and individual matches have
a one-second deadline. The longer match deadline is still asynchronous and
bounded, and covers an observed Windows scheduling delay while a 73,444-rule
standby generation compiled. Listener error paths fail open, and a worker
failure causes the latest ruleset to be loaded into a fresh worker on the next
operation.

One failure path intentionally fails closed. Electron 43 can crash its main
process when an `onBeforeRequest` callback redirects a CORS preflight: the
native preflight request has no target URL-loader client, but Electron's redirect
path dereferences it. Summer therefore converts `OPTIONS` redirects into
cancellation before returning the callback to Electron. This guard applies to
both browser-owned DNR decisions and blocking extension `webRequest` responses.
Other request methods still redirect normally, and response-header redirects
are unaffected. This is a bounded Chrome-parity deviation: the preflight fails
instead of following the extension redirect, preserving the blocker's privacy
intent without risking the whole browser process.

Current DNR support is intentionally narrower than Chrome. Static, dynamic, and
session rules use one browser-owned decision pipeline; supported conditions
include bounded URL, request, initiator, and top-frame domains, resource types,
request methods, response headers, and session-only Summer tab filters. Matching rules
can allow, block, redirect, upgrade schemes,
or modify request and response headers with deterministic conflict handling.
Redirects accept one validated direct URL, extension path, regex substitution,
or complete URL transform. Extension-controlled `urlFilter` and `regexFilter`
matching, `regexSubstitution`, and `isRegexSupported` operations compile through
the shared `re2js` adapter instead of JavaScript `RegExp`. DNR patterns are
ASCII-only; rules default to case-insensitive matching, while
`isRegexSupported` defaults to case-sensitive matching and honors
`requireCapturing`. Syntax failures and the Chrome-shaped 2 KiB compiled-program
cap report `syntaxError` or `memoryLimitExceeded` without executing an unsafe
pattern. Included response-header conditions use any-match semantics, excluded
conditions take precedence, and late block or redirect actions run only after
response headers exist. Extension service-worker DNR methods use either Electron's
implementation or Summer's authenticated relay. With
explicit `declarativeNetRequestFeedback` permission, `getMatchedRules`
returns bounded recent-match telemetry under a 20-call-per-10-minute quota.
Summer does not substitute a declared `activeTab` permission for a browser-owned
grant, and calls are not gesture-exempt until the relay can authenticate a user
gesture. `setExtensionActionOptions` validates live tab ownership atomically and
connects non-allow rule counts to the extension action badge. Rules outside
Summer's supported condition subset are skipped rather than partially applied.
Imported or custom uBO Lite filters that rely on unsupported DNR conditions are
not a complete parity target.

Summer's main process also observes request lifecycle events and forwards
Chrome-shaped `webRequest` details only to active extensions that declared the
permission and match both the request URL and its initiator through their host
permissions. Each listener's URL, resource-type, tab, and window filters are
enforced before delivery. Request headers, response headers, and upload data
are included only when that listener requested the corresponding
`extraInfoSpec`; listener removal also removes the browser-side subscription.
Blocking responses are resolved within Summer's bounded 500 ms deadline.

For normal Summer tabs, `webNavigation` exposes the top-level and OOPIF-safe
subframe lifecycle:
`onBeforeNavigate`, `onCommitted`, `onDOMContentLoaded`, `onCompleted`,
`onErrorOccurred`, `onReferenceFragmentUpdated`, and `onHistoryStateUpdated`.
Electron process/routing pairs receive stable Chrome-shaped frame IDs that are
shared by navigation queries, events, and authenticated user-script senders.
Listener-specific string filters and RE2 `urlMatches` and
`originAndPathMatches` filters are enforced through the shared linear-time
`re2js` adapter; as in Chrome, schemes and ports are ignored. CIDR filters are
rejected because Summer does not yet provide their exact semantics. Electron
does not expose Chrome's exact
transition cause, so committed and same-document events report `link` with no
qualifiers, and same-document event type is inferred from the old and new URL.
`onCreatedNavigationTarget` and prerender-specific `onTabReplaced` remain
unavailable. MV3 worker `webNavigation` subscriptions
are retained by Summer's bounded service-worker relay so registered navigation
events can wake a suspended worker when Electron omits that namespace.

Electron retains only the last native listener registered for a `webRequest`
event. Summer therefore installs one browser-owned `onBeforeSendHeaders` hook
per session and composes internal policies behind it in registration order.
Header changes flow to later policies, cancellation is preserved, and the
native callback is invoked exactly once. Internal request policies must register
through `electron/main/utils/webRequestListeners.ts` instead of attaching a
second native `onBeforeSendHeaders` listener.

Extensions with required `notifications` permission can call a limited
`chrome.notifications` implementation from extension pages and Manifest V3
workers. The supported methods are `create`, `update`, `clear`, `getAll`, and
`getPermissionLevel`; supported events are `onClicked`, `onClosed`, and
`onButtonClicked`. Body clicks, action-button clicks, and user or programmatic
closure remain distinct browser-owned signals. Notifications are displayed
through Electron's native OS surface after the main process verifies the sender
is an active managed extension with the required permission. `image`, `list`,
and `progress` templates are validated and flattened safely where the OS API has
no matching rich presentation; extension-owned images are realpath-contained
and size-limited. Action count, close reporting, urgency, and persistence still
vary by operating system.

Extensions with required `cookies`, `bookmarks`, `history`, `downloads`,
`debugger`, `fontSettings`, `idle`, `proxy`, `privacy`, `contentSettings`,
`browsingData`, and related permissions can use Summer's compatibility bridge
from extension pages. MV3 workers use Electron's native implementation where
complete plus Summer's authenticated service-worker relay for browser-owned
APIs and events. That includes cookies, bookmarks, history, downloads, alarms,
notifications, windows, context menus, web navigation, web-request events, and
a bounded 10 MiB `storage.session` area. When Electron supplies the complete
native area, Summer preserves it as the one authoritative owner, including
`setAccessLevel`, content-script isolation, and scoped `session.onChanged`
events. Content scripts are denied under `TRUSTED_CONTEXTS` and receive access
only after a trusted extension context selects
`TRUSTED_AND_UNTRUSTED_CONTEXTS`; the website's page world receives no bridge.
Summer's authenticated in-memory provider remains a fail-closed fallback for
trusted pages and workers if a future runtime lacks the complete native area.
Session values survive MV3 worker restarts in the current browser process and
are cleared when the extension reloads or unloads, or when Summer exits.
Worker cookies, notifications, alarms, popup-window mutation, and other
browser-owned surfaces no longer return fabricated empty or success values;
they call the authenticated main-process broker. Managed storage remains an
empty read-only area when no enterprise policy provider exists. An absent
global `browser` remains assignable so standard webextension polyfills can
alias it to `chrome`.
Cookie operations use Summer's real persistent browsing session. Bookmark URL
records remain authoritative in Summer's bookmark store, while a bounded,
persistent sidecar supplies Chrome-style folders, nesting, parent membership,
and sibling order. Corrupt or stale sidecar metadata is repaired without
deleting URL bookmarks, and unmapped bookmarks fall back to the bookmarks bar.
Creates, moves, updates, non-empty `removeTree()` operations, and folder events
use that hierarchy; browser-owned edits or Chrome/Safari imports continue to
deliver URL-bookmark lifecycle signals to pages or wakeable workers. Bulk
reorder events remain reserved for browser-owned UI sorting.
History operations use Summer's on-device visit store
populated from normal tab navigations. Download operations start real Summer
downloads, expose Chrome-shaped in-memory records, and route
pause/resume/cancel/icon/file operations to active Electron download items
where available. Debugger sessions attach only to HTTP(S) Summer tabs by
`tabId`, are owned by the exact extension that attached them, route Chrome's
allowed CDP domains and child-session identifiers, and deliver bounded
`onEvent` and `onDetach` signals to pages or wakeable workers. Tab closure,
navigation to a browser-owned URL, and extension unload clean up ownership.
Proxy, privacy, content-settings, and font-settings calls use browser-owned
profile state. Content-setting objects expose Chrome's `get`, `set`, and
`clear` contract without inventing `onChange` events that Chrome does not
provide. Idle queries use Electron's OS idle state rather than returning a
constant result.

Some desktop APIs are intentionally first-pass compatibility surfaces rather
than full Chrome UI parity. Desktop `commands` accepts Chrome's string or platform-keyed manifest
shortcuts, applies platform modifier rules, and dispatches non-reserved keyboard
combinations to `onCommand`; conflicts resolve deterministically by extension ID,
while active Summer shortcuts, AltGr combinations, and `_execute_action` remain
browser-owned. `contextMenus` stores menu
definitions, appends matching entries to Summer's tab context menu, and emits
click events back to extension pages and service workers. A click on a
persistent item wakes its unloaded MV3 worker, waits for `onClicked` to
subscribe, and then delivers the event once. `sidePanel.open()` hosts the
declared extension page in a sandboxed native side panel that reserves browser
layout space; one panel is active at a time, and `setPanelBehavior()` can route
toolbar action clicks to it. `omnibox` recognizes a manifest
keyword followed by a space,
requests at most eight suggestions, and dispatches selections back to the
extension.

Blocking `webRequest` responses are validated in the browser process and have
a 500 ms deadline. Offscreen documents are hidden, sandboxed extension pages;
they may use only `chrome.runtime`, and `AUDIO_PLAYBACK` documents close after
30 seconds without playback. `userScripts` is disabled per extension until the
user explicitly allows it in Extensions. Approved scripts run in bounded,
isolated worlds and may target only reviewed host permissions; `document_start` is injected at the earliest
top-level navigation-commit event available to Electron, and all-frame registrations run on
eligible frame events. `configureWorld()` applies a bounded custom CSP before
injection and can expose only the isolated world's own `chrome.runtime`
messaging surface. One-shot messages and long-lived ports are authenticated to
the exact extension, world, live frame, tab, current host grant, registration,
and explicit user approval before they reach `onUserScriptMessage` or
`onUserScriptConnect`.
Generic `identity.launchWebAuthFlow()` runs in a fresh in-memory, sandboxed
session and resolves only an exact extension redirect origin; Google-account
token APIs remain unavailable. Native messaging resolves a
platform-installed Chrome/Chromium host manifest, verifies its exact extension
allowlist and executable path, and uses bounded Chrome framing. One-shot calls
have a 30-second deadline; persistent `connectNative()` ports enforce global and
per-extension process limits, bounded pending writes, framed message limits,
authenticated ownership, and unload/shutdown cleanup.

Desktop and tab capture use a browser-owned bounded source picker and
Electron-issued media source IDs; no raw `desktopCapturer` or `WebContents`
object crosses into extension code. Tab capture streams are available only to
extension pages; `getCapturedTabs()` and `onStatusChanged` track pending,
active, stopped, and error transitions owned by that exact extension page.
Pending stream IDs expire after 30 seconds. Cancelling or leaving a desktop
source picker aborts the native dialog, while committed cross-document navigation,
renderer loss, extension unload, and browser shutdown release the exact
browser-owned capture records. Leaving the extension page also stops its live
media tracks and clears pending capture descriptors; same-document navigation
preserves the current capture owner.
The reported `fullscreen` flag remains `false` because Electron does not expose
the captured-tab fullscreen state to this broker. Tab groups
provide persistent-for-the-process Chrome metadata, membership, events, and
atomic same-window movement; cross-window group movement remains rejected.
Extension pages with `tts` permission receive the platform Web Speech provider
through Chrome-shaped callback and Manifest V3 Promise methods;
service workers do not have a speech engine and therefore do not expose TTS.

Compatibility initialization does not scan or rewrite installed extension
content. Its cost is bounded to the registered frame or service-worker preload
and the APIs requested by that extension context.

## Fallback inventory

Every compatibility substitution must be either brokered, deliberately local,
or explicitly unsupported. The registry is enforced by the central broker
before dispatch, so a Chrome-shaped member used for feature detection cannot
silently claim success. The intended endpoint column records the design target,
not a claim that the target is already scheduled or implemented. Chrome parity
is not the target where it would weaken Summer's permission or isolation model.

| Surface | Current behavior | Classification and reason | Intended endpoint | Source |
| --- | --- | --- | --- | --- |
| Page alarms, notifications, DNR, and action state | The generic broker wrapper installs before specialized APIs, and each setup step is isolated from failures in another namespace. | First-class Summer broker; one restricted Chromium namespace cannot suppress unrelated APIs. | Keep the composable broker, but require every exposed member to have one real owner, isolated initialization, and an explicit unavailable result when its owner cannot start. | `electron/preload/extensionApiAugment.ts` |
| Worker cookies, bookmarks, history, downloads, alarms, notifications, windows, and settings | Calls and events use the authenticated worker relay; cookie access also requires a matching HTTP(S) host permission. | First-class Summer broker. | Give pages and workers the same supported semantics through one authenticated, permission-checked relay, including worker wake-up, lifetime, cancellation, and cleanup. | `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| Runtime and external messaging without a complete native pair | Same-extension one-shot and port messaging use the authenticated browser-owned router, with bounded payloads, reply lifetimes, worker wake-up, disconnect, and extension cleanup. Extension-to-extension calls follow Chrome's manifest default: every extension may connect when `externally_connectable` is omitted, while a present key restricts callers to `ids`; one-shot messages dispatch through `onMessageExternal` and ports through `onConnectExternal`. Regular top-level HTTP(S) pages receive only a narrow `runtime.sendMessage()` method, and the browser process accepts it only when the target's validated `externally_connectable.matches` covers the authenticated current page URL. | First-class Summer broker; page-owned values select a target but never grant authority, child frames and private-profile pages cannot use the website bridge, and no page-owned transport carries browser API authority. | Retain browser-owned sender authentication, bounded replies and worker wake-up while expanding external message surfaces only when their manifest declaration and profile boundary can be enforced exactly. | `electron/preload/externalExtensionMessaging.ts`, `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| Optional permissions | Required permissions and hosts are active immediately; `optional_permissions` and `optional_host_permissions` remain inactive until `permissions.request()` receives explicit native user consent. Exact and wildcard-covered narrower host grants persist per extension, affect browser-process API and host authorization, can be revoked through `permissions.remove()`, emit wakeable `onAdded` / `onRemoved` events, and are erased when the extension is removed. | First-class browser-owned consent boundary; undeclared requests, required-permission removal, malformed input, and denied prompts fail without changing grants. | Keep grants constrained to the currently declared optional set, serialize concurrent mutations, and require browser-owned consent before widening extension authority. | `electron/main/extensions/extensionChromeApis.ts`, `electron/preload/extensionApiShim.ts` |
| Alarms | Bounded alarms persist in browser-owned storage, are restored only for active managed extensions that retain the effective permission, fire overdue one-shots after restart, and advance repeating schedules before dispatch. | First-class persistent scheduler with per-extension quotas and lifecycle reconciliation. | Retain durable mutation-before-event ordering, worker wake-up, and shutdown-safe timer cleanup. | `electron/main/extensions/extensionAlarms.ts` |
| Tabs, frame navigation, and general read-only services | `tabs.move` transfers individual tabs within or between regular Summer windows of the same privacy class, emits `onDetached` / `onAttached`, and reconciles both tab strips; per-origin zoom methods/events, OOPIF-safe frame-tree webNavigation, MHTML page capture, non-private recently closed tabs, top sites, and bounded CPU/memory/storage information use browser-owned state. | First-class where Summer owns an exact source; grouped cross-window movement and unsupported navigation target events remain explicit gaps. | Add a surface only when its identity, ownership, and lifecycle can be represented without inventing Chrome state. | `electron/main/front/tabs.ts`, `electron/main/extensions/extensionBrowserHost.ts`, `electron/main/extensions/extensionWebNavigation.ts`, `electron/main/extensions/extensionPageCapture.ts`, `electron/main/extensions/extensionSessions.ts`, `electron/main/extensions/extensionSystemInformation.ts` |
| Capture, tab groups, and TTS | Desktop/tab capture uses a browser-owned bounded picker and Electron media IDs; pending/active/stopped/error tab-capture state is authenticated and observable through `getCapturedTabs()` and `onStatusChanged`; tab groups own same-window metadata and events; extension-page TTS uses Web Speech. | Bounded compatibility surfaces; captured-tab `fullscreen` remains false, cross-window groups remain unavailable, and worker TTS has no speech engine. | Add only state Summer can observe exactly, and keep raw Electron capture authority out of extension contexts. | `electron/main/extensions/extensionCapture.ts`, `electron/main/extensions/extensionTabGroups.ts`, `electron/preload/extensionApiShim.ts`, `electron/preload/extensionTts.ts` |
| Native messaging | Chrome/Chromium native-host manifests and allowlists are validated before one-shot or persistent framed processes start; ownership, process count, writes, replies, and teardown are bounded. | First-class broker for platform-installed compatible hosts; host installation/discovery UI is outside Summer. | Retain exact executable and origin validation and never expose child processes or filesystem paths to extensions. | `electron/main/extensions/extensionNativeMessaging.ts` |
| `storage.session` | Summer preserves Electron's complete native MV3 area, including default content-script denial, trusted-page grant/revocation, shared values, and scoped change events. A complete native area must expose every promised method plus `onChanged`; otherwise trusted pages and workers receive Summer's authenticated in-memory fallback while content-script access fails closed. | First-class native Chromium binding on the pinned runtime, with a bounded Summer fallback for trusted extension contexts. | Keep Chromium as the authoritative isolated-world owner while it passes the completeness and E2E contract; never replace it with a DOM relay or manifest-widening bootstrap. | `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionSessionStorage.ts`, `playwright/extensions-manager.spec.ts`, `docs/chrome-extension-content-script-session-storage.md` |
| Worker `storage.sync` | A browser-owned device-local provider enforces Chrome-shaped item, byte, and write quotas and emits scoped change events, but has no cross-device transport. | Deliberate, explicitly reported substitution; Summer has no authenticated cross-device sync service. | Use an authenticated, encrypted cross-device sync provider with conflict and quota semantics; until one exists, report the device-local substitution explicitly rather than presenting it as real sync. | `electron/preload/extensionApiShim.ts` |
| Worker `storage.managed` | A validated read-only OS-admin policy file supplies extension-scoped values and change events; the area is empty when no policy file exists. | First-class bounded policy provider with an accurate empty state when no administrator source is configured. | Connect to a validated read-only policy provider and emit policy change events; an empty area remains correct only when no policy source is configured. | `electron/preload/extensionApiShim.ts` |
| Context menus without an authenticated relay | Eligible extension pages and workers use the authenticated context-menu host with persistent worker wake-up and extension cleanup; unsupported contexts keep the namespace absent. | First-class Summer broker where an authenticated relay exists, otherwise fail closed. | Route every eligible page and worker through the authenticated context-menu host with worker wake-up and cleanup; otherwise keep the namespace absent. | `electron/preload/extensionApiShim.ts` |
| `idle` | Queries use the OS idle state, while one shared browser-owned monitor emits threshold-aware `onStateChanged` transitions and cleans up with the extension lifecycle. | First-class cross-platform OS-backed query and event producer. | Add a cross-platform OS-backed transition producer for `onStateChanged`, with one shared poll/subscription source and extension lifecycle cleanup. | `electron/main/extensions/extensionChromeApis.ts` |
| Browsing-data removal | Cookies, Summer history, and Summer download records honor their supported scopes; cache rejects unsupported scoping. Password and autofill-profile removal is all-time and profile-wide, preflights every requested category, names the requesting extension and categories in a localized native warning, rejects cancellation, verifies password removal, and invalidates open Settings state. Payment cards are never deleted. | Honest bounded subset with preflight validation and explicit user confirmation for sensitive profile-wide deletion; no false success for cancellation or data Summer cannot prove it erased. | Implement each category only with the exact scope Summer owns; retain confirmation for password/profile deletion, keep payment cards out of `formData`, and reject unsupported or scoped requests before any deletion. | `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionBrowsingData.ts`, `electron/main/credentialStore.ts`, `electron/main/autofill.ts` |
| Blocking `webRequest` | Event-specific filters and extra-info options are validated in the browser process; protected request and response headers require `extraHeaders`, blocking results are authenticated to their exact extension context, terminal DNR actions run before `webRequest`, DNR header changes retain priority, cross-extension conflicts use stable extension-ID ordering, and `handlerBehaviorChanged()` has Chrome's 20-call-per-10-minute quota. | First-class bounded broker; unsupported schemes or fields reject, responses fail open after 500 ms, and protected headers that were withheld from a listener survive full-list replacements. | Add resource types, schemes, and security information only when Electron can identify and expose them without widening the listener's authority; retain authenticated responses, deterministic conflicts, and bounded timeouts. | `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionDeclarativeNetRequest.ts` |
| DNR rules, match feedback, and action options | One bounded browser-owned decision combines separately compiled static and mutable matchers; active/standby worker generations keep the previous rules serving until a replacement is complete. Deprecated initiator aliases and top-frame domains use authenticated request context, session rules honor Summer tab filters, request-method exclusions work across scopes, validated direct, extension-path, regex, and full URL-transform redirects are supported, response-header include/exclude conditions can make late block or redirect decisions, request and response header actions use per-extension priority with stable extension-ID ordering, before-request decisions select one candidate per extension and resolve block before redirect or upgrade before allow, and feedback returns a five-minute match history under Chrome's 20-call-per-10-minute quota, matched header rules are included, action-count options reject missing tabs or disabled badge updates atomically, allow rules do not inflate the count, and Chrome-shaped static-ruleset, mutable-rule, unsafe-rule, and regex-count limits are enforced atomically under Summer's stricter 150,000-static-rule cap. Extension-controlled patterns use the shared linear-time `re2js` engine with ASCII validation, capture-aware substitution, and a Chrome-shaped 2 KiB compiled-program cap. `allowAllRequests` priority is retained across authenticated frame hierarchies and cleared on navigation, tab removal, or extension changes. | First-class bounded broker; unsupported, unsafe, or incorrectly scoped rules reject instead of being widened. | Add new resource types only when Electron can identify them; keep RE2 syntax and compiled-program compatibility covered when either Chrome or `re2js` changes. | `shared/chromeDeclarativeNetRequest.ts`, `electron/main/extensions/extensionDeclarativeNetRequest.ts` |
| `identity` account/OAuth methods | Generic web-auth flows use a fresh nonpersistent session with downloads, permissions, popups, webviews, Node, and DevTools denied; only the authenticated extension's exact `chromiumapp.org` redirect resolves. Redirect URL calculation and empty local profile info remain available. Google-account token calls still reject. | First-class isolated generic OAuth flow; browser-account token APIs require an account provider Summer does not have. | Add token APIs only with a user-consented browser account provider, scoped token storage, revocation, and clear account UI. | `electron/main/extensions/extensionIdentity.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| `offscreen` | Desktop: one hidden sandboxed document per extension supports Summer's reviewed Chrome reason set, authenticated runtime messaging, audio inactivity, and crash/load cleanup. Mobile V56: required-permission MV3 callers get Promise-only create/close/has for exact `WORKERS`, one PAGE document per extension/eight globally, a six-member provisional runtime plus non-enumerable `lastError`, verified package modules/Workers, and process-only teardown under strict CSP/navigation/network bounds. Other mobile reasons and suspended/killed-app execution are absent. | First-class bounded hidden-document hosts with deliberately different reviewed reason/lifecycle sets. | Expand a mobile reason only with reason-specific sandbox and lifetime enforcement on both native hosts; retain exact package/document ownership, cleanup, and honest process-lifecycle limits. | `electron/main/extensions/extensionOffscreenDocuments.ts`, `packages/summer-android/src/extensions/mobileExtensionPackage.ts`, `packages/summer-android/android/app/src/main/java/com/hiketech/summerbrowser/SummerTabsPlugin.java`, `packages/summer-android/ios/App/App/SummerTabsPlugin.swift` |
| `userScripts` | Explicitly user-approved registrations execute in isolated worlds on reviewed hosts. A frame-preload handshake gates every injection, installs each configured CSP before execution, and exposes only permission-bounded runtime one-shot/port messaging when `messaging` is enabled. | First-class bounded isolated-world bridge; every call is re-authorized against the live frame, current host grant, matching registration, extension state, and user approval. | Retain fail-closed authorization, bounded messages/ports, and configuration-before-injection ordering across every injection phase. | `electron/main/extensions/extensionUserScripts.ts`, `electron/preload/extensionUserScriptWorld.ts`, `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionManager.ts` |
| Desktop `commands`, `omnibox`, and `sidePanel` | On desktop, string or platform-keyed shortcuts are normalized with Chrome's modifier rules, active Summer accelerators and AltGr combinations remain unavailable, conflicts resolve by stable extension ID and appear unassigned through `getAll()`, omnibox deletion/disposition events are complete, and default or tab-specific native panel state is wired into Summer UI; tab panels hide and restore with tab activation. | First-class Summer integrations with documented bounds rather than Chrome UI emulation. | Add a user-facing browser-owned shortcut remapping surface before claiming Chrome's shortcut-management UI; retain deterministic conflicts, native panel state, and consistent keyboard and accessibility behavior. | `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionSidePanels.ts`, `ui/store/suggestions.ts` |
| Native messaging | `sendNativeMessage()` and `connectNative()` discover platform-installed Chrome/Chromium manifests, require the exact extension origin in `allowed_origins`, canonicalize the executable, and launch without a shell. One-shot calls use bounded framing and a 30-second deadline; persistent ports authenticate ownership, enforce global/per-extension process and pending-write limits, deliver bounded framed messages, and terminate on disconnect, extension unload, or shutdown. | First-class bounded transport with strict host ownership, framing, backpressure, and lifecycle cleanup; native-host installation and discovery UI remain platform-owned. | Retain authenticated ownership and guaranteed process termination while keeping host discovery limited to compatible Chrome/Chromium registrations. | `electron/main/extensions/extensionNativeMessaging.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| File-scheme access | Disabled by default. For an extension whose reviewed host declarations include file URLs, the manager can grant or revoke access; Summer reloads the extension with the matching Electron `allowFileAccess` setting and the authenticated status method reports the active grant. | Explicit browser-owned per-extension grant tied to reviewed manifest access. | Keep file access disabled by default, require an explicit user decision, revoke stale grants when reviewed file-host access disappears, and never infer the grant from a host declaration alone. | `electron/main/extensions/extensionManager.ts`, `electron/main/extensions/extensionChromeApis.ts`, `electron/preload/extensionApiShim.ts` |
| Incognito access | Disabled per extension by default. Extensions Manager can explicitly allow or revoke access; `isAllowedIncognitoAccess()` reports that browser-owned grant. Allowed extensions are loaded separately into the disposable private session while at least one private window exists. | Bounded split runtime. Electron requires a path-backed persistent session to load an extension, so Summer uses a random temporary directory, unloads extensions, clears Chromium data, attempts whole-directory deletion after the last private window, and reclaims locked dead-process leftovers at startup. Reuse is blocked only when neither logical clearing nor physical removal succeeds. | Retain explicit per-extension consent and isolated lifecycle cleanup; adopt a true in-memory extension host if Electron provides one. | `electron/main/extensions/extensionManager.ts`, `electron/main/extensions/privateExtensionSession.ts`, `electron/main/front/privateSessionStorage.ts`, `electron/main/front/sessions.ts` |
| IDs without `crypto.randomUUID` | Browser or Node UUIDs are preferred; supported older contexts use `crypto.getRandomValues`, and identifier generation rejects when secure randomness is unavailable. | Cryptographically random but still non-authoritative identifiers. | Use browser-provided UUIDs in every supported context, or a cryptographically random browser-owned fallback; identifiers remain non-authoritative. | `electron/preload/extensionApiShim.ts` |

## Security boundary

The user-facing manager is a permanent, bundled Summer App. Its page has no
Electron, filesystem, or raw IPC access. The app receives a narrow browser-owned
capability that accepts only opaque record and one-use confirmation tokens:

```text
Extensions Manager Summer App
        |
        | select / review / confirm / enable / disable / reload / options / userScripts approval / remove
        v
Browser-owned extension service
        |
        v
persist:summer Session.extensions
        |
        | explicit per-extension private grant
        v
random disposable private Session.extensions
```

The browser process owns the native file and directory pickers, downloads,
package extraction, manifest validation, permission fingerprint, persistence,
and Electron calls. A selected manifest is size-limited and parsed before any
extension code runs. Confirmation re-reads the manifest to detect
declared-access changes during review, and extensions are loaded with local
`file://` access disabled.

CRX2 and CRX3 developer signatures are verified before extraction. A Web Store
install is downloaded only from Google's official HTTPS update service and the
signed CRX identity must equal the requested extension ID. Transient update
service failures are retried with bounded response size and inactivity limits.
Summer retains that verified Web Store identity and checks for a newer signed
package five minutes after the extension runtime starts and every five hours
while it remains active. A strictly newer package with the same or reduced
declared access is applied automatically. If the update adds any required or
optional API, host, or content-script match declaration, Summer unloads the old
version, stages the immutable signed package, and requires the existing browser-
owned access review before loading the update. A failed download, invalid
package, incompatible manifest, same-version response, or downgrade leaves the
installed version unchanged. After a replacement commits and loads, Summer
removes the superseded browser-owned package; uninstall also removes the current
and any staged package. Local CRX and unpacked-source updates remain explicit
user actions.
Summer writes the verified public key into the managed manifest so Electron
retains the signed Web Store identity, and rejects a package whose existing
manifest key conflicts with its signature. ZIP paths, symbolic
links, unsupported compression, invalid CRCs, excessive file counts, and
bounded package or extracted-size violations are rejected. Summer currently
accepts packages up to 512 MiB, CRX extraction up to 1 GiB, and unpacked
snapshots up to 512 MiB so large store extensions still install without
removing ZIP-bomb guardrails. Package
files are extracted into Summer's browser-owned profile storage; the original
local CRX is never executed in place.

Unpacked folders are copied into immutable browser-owned snapshots before the
review is confirmed. Electron loads the reviewed snapshot, not the mutable
source folder, so an installed extension keeps working if the original source
folder moves or disappears. Selecting the same source folder again creates a
new snapshot and is treated as an update; expanded declared access requires a
new permission review before the snapshot replaces the installed copy. Snapshot
replacement is atomic, and obsolete browser-owned snapshots are cleaned without
touching user files.

Extension-owned pages do not inherit Summer's website permission defaults or
website preload capabilities. Summer's registered session preload exits before
installing `window.summi`, autofill probes, overlay IPC, or browser key
propagation on `chrome-extension://` pages. Extensions are never loaded into
Summer's browser-interface session.
## Browser-owned API lifecycle

Summer's browser-owned Chrome API host normalizes tabs and windows before they
cross the extension boundary. It supports tab lookup, query, creation, update,
reload, removal, and the corresponding create/update/activate/remove events.
Tab queries support URL patterns and normal/popup window filtering. Extension
pages can open their declared options page with `runtime.openOptionsPage()`.
Window reads and focus/removal events cover only windows in the current Summer
profile process. An extension may create one same-extension page in a popup
window and later focus or resize that popup; it may focus, but not resize, a
normal Summer browser window. Individual tabs may move between regular windows
of the same privacy class with Chrome-shaped detach/attach events; moving a
whole tab group across windows, cross-extension popup navigation, and arbitrary
browser-window mutation remain rejected explicitly.

Toolbar action state is live rather than manifest-only. `action` and the MV2
`browserAction` alias support default and per-tab titles, popups, icons, badges,
and enabled state. Clicking an enabled action without a popup wakes the
extension's service worker and emits `onClicked`; state is discarded when its
tab or extension is removed.
## Current limits

- Automatic discovery applies only to extensions installed from the Chrome Web
  Store. Local CRX and unpacked extensions must still be updated explicitly;
  newly declared access in any update always requires another review.
- Electron natively supports only a subset of Chrome extension APIs. Summer's
  compatibility bridge fills many missing namespaces, but some behaviors remain
  partial until their Summer UI surfaces or platform services are implemented.
- Private access is disabled independently for every extension until the user
  allows it in Extensions Manager. The private host deliberately receives
  Electron's native extension surface, not Summer's regular-profile API
  augmentation: regular-profile bookmark, history, cookie, download, and other
  browser-owned brokers must not become reachable from a private extension
  context merely because the same extension is enabled normally. Compatibility
  features supplied only by those Summer brokers therefore remain unavailable
  in private windows.
- Summer validates the manifest `incognito` mode. `not_allowed` disables the
  private-access control and cannot be overridden; both Chrome's default
  `spanning` declaration and an explicit `split` declaration execute through
  Summer's isolated split private host, which is reported in manager state.
- Electron cannot load extensions into a nonpersistent in-memory session. The
  private host is consequently path-backed in a random temporary directory.
  Summer clears that directory and attempts deletion after the last private
  window. Files Chromium keeps locked are retried on later lifetimes and startup;
  crash recovery and directory deletion are best effort rather than
  cryptographic secure erasure.
- `storage.sync` remains a quota-compatible, device-local provider. True sync
  requires an authenticated and encrypted cross-device service, identity and
  recovery UX, conflict resolution, and compatible quota semantics; Summer has
  none of that infrastructure today, so it does not present the local provider
  as cross-device synchronization.
- Method- and event-level facts for every callable or subscribable member
  exposed by Summer's compatibility shim live in
  `shared/chromeExtensionCompatibility.ts`. The registry records runtime
  ownership, manifest versions, supported contexts, and precise limitations.
  Contract tests compare that registry recursively with the runtime shim.
  Absence from the registry is not a support claim, and harmless worker-side
  placeholder shapes are explicitly marked unsupported.
- DevTools extension pages and blocking `webRequest` listeners are accepted
  because the current Electron extension host lists them as supported. Electron's
  own `session.webRequest` handlers still take precedence over extension
  handlers when both match a request.
- Mac App Store builds now attempt extension support. Store review may still
  reject dynamically loaded extension code or require a reduced distribution
  variant; that decision belongs to release validation rather than runtime
  feature hiding.

See Electron's
[Extensions API documentation](https://www.electronjs.org/docs/latest/api/extensions)
for the host runtime's compatibility policy.

## Development and verification

The app source lives in `packages/extensions-manager/`. Its generated built-in
release is produced with:

```sh
npm run build:sap -- extensions-manager
```

Use a disposable Summer instance for manual checks. Never load development
extensions into a real browser profile during automated testing:

```sh
electron . --instance-id=extensions-test --instance-data-dir=/absolute/disposable/path
```

The repository includes a broad local API probe extension at
`examples/chrome-extension-api-probe/`. Load that folder through the Extensions
Manager in a disposable Summer instance to exercise the service-worker bridge,
cookies, bookmarks, history, downloads, commands, context menus, side panel,
omnibox, identity, idle, font settings, offscreen documents, user scripts,
proxy/privacy/content settings, browsing data, debugger, capture, tab groups,
native messaging, system information, page capture, sessions, top sites, TTS,
and web-navigation
compatibility surfaces.

`playwright/popular-extensions-live.spec.ts` is an opt-in live compatibility
harness for uBlock Origin Lite, Adblock Plus, Tampermonkey, Dark Reader,
Bitwarden, Grammarly, MetaMask, and React Developer Tools. It installs each
selection into a short disposable profile, verifies the signed runtime ID,
starts its Manifest V3 worker, opens its declared popup, and fails on crashes
or unexpected high-severity extension console errors. uBlock Origin Lite and
Adblock Plus must expose enabled native static DNR rulesets. The bundled API
probe must receive a popup-to-worker runtime response, install a bounded session
DNR rule that blocks a loopback request, and restore the request after removing
the rule. Dark Reader must visibly transform a controlled light loopback page;
Grammarly must attach its account-free editor integration to a controlled text
field; and React Developer Tools must install its Fiber-capable page-world hook.
Bitwarden must render its packaged signed-out vault entry actions, while
credential autofill remains outside this account-free probe because it requires
a disposable account and populated vault. MetaMask must complete
disposable-wallet onboarding, inject a responsive EIP-1193 provider into a
loopback site, approve the site's account request, approve a personal signature,
expose and reject a transaction request, and restore the same wallet and approved
account after restarting the disposable profile. The harness never records a
recovery phrase or uses a funded account.
Extensions
without a deterministic probe are reported explicitly as lifecycle-only rather
than functionally supported. Select one or more IDs through
`SUMMER_LIVE_EXTENSION_E2E`; use `SUMMER_LIVE_EXTENSION_CRX_PATH` or
`SUMMER_LIVE_EXTENSION_DIRECTORY` for a single local package. Set
`SUMMER_LIVE_EXTENSION_KEEP_PROFILE=1` only when retaining a failed disposable
profile for diagnosis.

The 2026-08-03 live Web Store compatibility pass completed all eight listed
extensions in 6.5 minutes through signed installation, worker startup, action
popup opening, and the extension-specific checks in the harness, without an
unexpected high-severity error. The tested downloads included uBlock Origin
Lite 2026.729.1529, MetaMask 13.41.0.0, and Adblock Plus 4.42.1. uBlock and
Adblock Plus both proved that their enabled static DNR rules were present and
blocked a live request; the probe records rule presence, block outcome, and
elapsed time so a closed extension page cannot be mistaken for a successful
block. uBlock's complete popup UI stabilized on repeated launches in 335-649 ms
in that run. A separate deterministic regression proves its DNR-backed badge
state survives startup. Tampermonkey's programmatic toolbar icon, Bitwarden's
expected optional-native-messaging rejection, and the remaining popup/worker
lifecycles also passed. That historical run predates the deterministic Dark
Reader, Bitwarden signed-out UI, Grammarly editor-integration, and React
Developer Tools hook requirements above; those probes must pass in the next
opt-in live matrix before newer feature-level compatibility is claimed.

MetaMask provided a useful hostile-runtime regression. In three pre-fix Summer
runs its LavaMoat scuttling exposed missing `TextEncoder`, `structuredClone`, or
`Blob` access in Summer's late callbacks, while the same signed package ran
three times in stock Chromium without those errors. Summer now captures the
required browser primitives inside its isolated preload before extension code
can scuttle page globals. The same package then passed three out of three Summer
runs, and a unit regression deliberately scuttles those globals after preload
installation. This preserves the isolated bridge; it does not expose Node or a
new page-world capability.

The opt-in DNR benchmark uses the observed uBlock Origin Lite rule shape: 18,361
enabled rules, including 17,324 URL filters. Run the indexed comparison first,
then build the Electron entries before running the real worker-client check:

```powershell
$env:SUMMER_DNR_BENCHMARK = "1"
npm run test:unit -- tests/declarativeNetRequestBenchmark.test.ts `
  --disableConsoleIntercept
Remove-Item Env:SUMMER_DNR_BENCHMARK

npm run build:vue
$env:SUMMER_DNR_WORKER_BENCHMARK = "1"
npm run test:unit -- tests/declarativeNetRequestBenchmark.test.ts `
  --disableConsoleIntercept
Remove-Item Env:SUMMER_DNR_WORKER_BENCHMARK
```

The 2026-08-01 Windows investigation measured the original flat matcher at
133.19 ms per request. Preparing the request once reduced that linear reference
to about 86.71 ms; the conservative index reduced it to about 0.05 ms. Through
the production worker client, 200 indexed decisions averaged 0.11-0.12 ms
round-trip while the calling event loop continued to tick during 1.8-3.8 seconds
of worker-side ruleset compilation. The worst observed caller-side scheduling
delay was 129 ms rather than several seconds. These numbers are
machine-dependent; the fixture and event-loop assertion are the reproducible
contract.

The 2026-08-03 replacement stress test starts a 73,444-rule static compilation
on the standby worker while sending a decision through the active worker. The
old generation continued to block the request, the Electron caller's event-loop
tick continued to run, and the observed active decision completed in 441.6 ms.
That measurement is why production uses a one-second asynchronous match budget
instead of the former 250 ms budget: the old limit failed open during heavy
compilation and briefly let a request escape. After the atomic swap, the
benchmark restores the 18,361-rule uBlock-shaped fixture and verifies 200
indexed decisions. The reproducible contract is continued active-generation
enforcement and caller responsiveness, not the machine-specific timing.

### Electron 43 preflight redirect crash proof

The 2026-08-02 Windows investigation reproduced browser-process access
violations while uBlock Origin Lite redirected a CORS `OPTIONS` request to its
`web_accessible_resources/noop.txt`. Symbolizing a full dump with the exact
Castlabs Electron 43.0.0 x64 private PDB produced this stack:

```text
network::mojom::URLLoaderClientProxy::OnReceiveRedirect
electron::ProxyingURLLoaderFactory::InProgressRequest::ContinueToBeforeRedirect
electron::ProxyingURLLoaderFactory::InProgressRequest::HandleBeforeRequestRedirect
```

The crashing `URLLoaderClientProxy` receiver was null. The owning request had
`for_cors_preflight_ = 1` and `current_request_uses_header_client_ = 1`, proving
that the DNR worker and JavaScript matcher were not the crash site. Electron's
43.0.0
[`ProxyingURLLoaderFactory`](https://github.com/electron/electron/blob/v43.0.0/shell/browser/net/proxying_url_loader_factory.cc)
constructs the special preflight request without a target client, then its
header-client redirect branch reaches `ContinueToBeforeRedirect`, which calls
`target_client_->OnReceiveRedirect(...)` without the preflight guard used by a
later headers-received path. The same unguarded source shape remained in
Electron 43.2.0 and the inspected upstream `main` snapshot. The focused runtime
regression test proves Summer now returns `{cancel: true}` for both DNR and
blocking-`webRequest` preflight redirects while retaining normal redirects for
other methods.

Summer retains `re2js` rather than adding the native `re2` addon because regex
execution is no longer a measured request-path bottleneck after indexing. A
native addon would add Electron/Node ABI rebuilds and cross-platform packaging
and signing obligations without evidence of a user-visible gain. The shared
pattern adapter keeps that backend replaceable if a future profile identifies
regex execution as the dominant indexed cost.
