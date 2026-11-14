# Chrome extension compatibility

> **Current mobile security boundary:** the shared manager can download, verify,
> review, persist, update, disable, and remove arbitrary signed CRX3 packages.
> Android executes compatible MV3 service workers and ordered MV2 background
> scripts in an isolated standalone QuickJS service, applies its explicitly
> advertised request-stage DNR subset (including Chrome 145 browser-attributed
> `topDomains` / `excludedTopDomains`, bounded verified package-resource
> substitution, and credential-free top-level `GET` direct URL/component/regex-
> substitution redirects)
> and resource-free declarative CSS, hosts
> verified local extension pages, and exposes bounded MV3/MV2 immediate
> JavaScript through authenticated execution-only worlds in the existing System
> WebView. MV3 calls default to isolated execution; explicit `MAIN` calls are
> confined to the authenticated current top frame and project no extension or
> native API into the website. Signed manifest and dynamic isolated content
> JavaScript runs automatically in authenticated per-extension worlds with only
> `runtime.id`, same-extension `runtime.sendMessage`, and scoped
> `runtime.lastError`. QuickJS is not a page renderer, and Summer does not bundle
> Chromium, Gecko, or another browser stack. Declarative MAIN scripts, exact
> pre-page-script `document_start`, complete content APIs/CSP isolation,
> subframe/subresource URL and regex-substitution redirects, package-document
> redirects outside the authorized top-level HTML path, header mutation, and
> private execution remain fail-closed; iOS work is currently paused. Mobile
> V46–V56 runtime sections below are historical
> implementation records, not current execution support. See
> [Android QuickJS extension runtime](android-quickjs-extension-runtime.md).

Summer can install signed CRX packages, download Chrome Web Store packages, or
load unpacked Chrome extensions through the bundled **Extensions Manager**
Summer App. This compatibility layer is built on Electron's supported
extension APIs; it does not provide full Chrome compatibility.

Desktop package downloads and unpacked snapshots publish completed directories
into browser-owned storage. Retrying an identical download can reuse an existing
directory only when its full content hash matches and it is an ordinary directory
inside that storage root. This also handles Windows' `EPERM` response to a
directory collision, including retrying after an abandoned permission review.
An unreadable, linked, or changed cached directory fails without replacing it;
signature verification and permission review still run for every install.

The namespace expansion and its remaining implementation/verification work are
tracked in [the namespace implementation record](chrome-extension-namespace-implementation.md).
That record distinguishes native APIs, implemented compatibility subsets, shared
type declarations, and platform services; namespace presence alone is not parity.

Desktop runtime update checks have a main-process coordinator and prepared
staging and consumption paths. The production package adapter verifies the
download's CRX signature and requested ID, hashes the transformed staging files,
and independently checks the final browser-owned directory before issuing an
in-memory receipt. Reused cache directories must match the complete file and
manifest digests; a cache path or saved JSON cannot recreate that receipt. File
reads have size, descriptor, path and directory checks, including rechecks of
earlier files. This detects ordinary concurrent changes and does not promise
atomic or tamper-proof execution against a privileged local process. A retained
signed archive can rebuild a restored stage offline, with fresh signature,
requested-ID and file checks and a new in-memory receipt. Serialized archive
references provide no authority. An earlier Windows package-publication failure
remains unexplained; subsequent focused passes do not establish its repair.

The older automatic updater also retains access-neutral candidates in
`deferredWebStoreUpdate` until reload or profile startup. These legacy records
contain a retained path and hash rather than the strong runtime-update receipt
or signed archive provenance. While such a legacy candidate is pending,
`runtime.requestUpdateCheck()` rejects; it does not promote that saved metadata
into verified update authority. This is a bounded compatibility limitation of
the combined paths, not complete parity between them.

Updates needing additional access or changing private-session declarations keep
the accepted incumbent code and grants until explicit review. Disabling and
re-enabling that incumbent leaves the candidate pending. Review binds the exact
installation, candidate and current settings; approval does not enable private
access. A reviewed update accepted while disabled remains disabled, with its
update lifecycle fact deferred until a healthy load. The manager controls and
disclosure have focused unit and English/Hebrew/Arabic browser UI checks using
controlled capabilities. Windows regular-profile MV3 pages and workers also
pass a real signed-update browser test for `requestUpdateCheck` and
`onUpdateAvailable`: shared discovery, throttling, full raw manifest events,
neutral staged reload, expanded-update review/cancellation, incumbent reload
and explicit approval. Check results use canonical versions; event manifests
retain their raw spelling. The test controls only the two fixture IDs' signed
download bytes, while Summer performs signature, installation and permission
checks. It does not verify the live Chrome Web Store service.

The same test verifies inherited, synchronous `runtime.getVersion()` through
both aliases and held/current references in pages and workers, including a real
second extension install and subsequent updates. The three owned update members
also retain their identities across that second install. This confirms existing
native support; it is not a third newly implemented API. Main-process update
facts retain their original document/logical-worker transport audience. Local
JavaScript callbacks are selected when that transport delivers the fact; a late
listener check proves no subscription replay, not a per-callback commit snapshot.
Actual cold-worker wake, offline restart, private-positive update APIs, MV2 and
other desktop platforms remain unverified. The earlier package-publication
failure and a separate file-reader timeout remain unexplained despite later
focused and browser passes.

A historical Windows MV3 probe, before the native-role and sidebar integration, of inherited `runtime.getContexts()` returned the
same single `BACKGROUND` context through both aliases in a real page and worker.
It omits the open extension tab and its two distinct child documents, whose
native frame and public tab/window IDs were independently observed. That probe
therefore failed: background-only enumeration does not establish full context
support. Other context roles and filters remain unverified. Complete support
requires native ownership integration; JavaScript-generated context/document
IDs are not a substitute.

The prepared Main creation path now carries a closed native role for ordinary
tabs and public popup-window tabs (`TAB`), toolbar action popups (`POPUP`), and
side panels (`SIDE_PANEL`). The matching SDK must consume that role before its
extension observer; an already initialized context cannot be relabeled during
adoption. Older SDKs without the optional native role witness retain existing
browser behavior, without claiming completed context enumeration. Public tab
registration uses each actual signed-32-bit `WebContents.id`; a Main-owned
registered-kind/Session/owner check selects messaging routes instead of an ID
range. Source and controlled-unit checks cover these changes; native creation,
clone/child initial-world behavior, DevTools ownership and explicit private
context classification remain separate verification gates.

Desktop `management.get` and `getAll` query Summer-managed extensions with retained
authenticated IDs. Real Windows regular-profile MV3 pages/workers pass native
refresh, optional grants, disabled-state restart, re-enable and removal checks
through held/current methods and Promise/callback forms. The cached native
namespace retains Summer's methods; shared runtime aliases keep callback errors
visible through the public runtime. Native `getSelf` is unchanged. Private
inventory and immutable native-cache shapes are unsupported; MV2 and other
desktop platforms remain unverified. See the implementation record for metadata,
permission boundaries, the accepted evidence and preserved earlier failures.

Desktop `management.onInstalled`, `onUninstalled`, `onEnabled`, `onDisabled`, and
`sessions.onChanged` also have Windows regular-MV3 page/worker acceptance. Native
load/unload facts and accepted manager changes carry historical safe metadata;
uninstall carries only the ID. The in-memory closed-tab journal emits zero-data
changes after committed mutations. Page-local event facades preserve callback
identity through contextBridge, native binding refresh and both aliases. Recipient
permission, installation and listener lifetimes are checked across queued worker
wake, including removal of one's own listeners after permission revocation.
Private-source exclusion is tested; allowed-private observers, MV2, other
platforms, durable/window/synced sessions and signed automatic updates remain
unverified. See the namespace record for exact scope and evidence.

Desktop `publicSuffix` supplies synchronous full-PSL domain/suffix lookups with
current main-owned permission checks, including retained methods after optional
permission revocation. Default/punycode IDNs work. Safe Unicode display now has
authenticated page/worker source wiring and a prepared private Chromium
spoof-aware formatter patch, but the installed binary lacks that native method:
non-ASCII `encoding: "display"` still rejects. Native compilation and positive
Unicode/spoof display acceptance remain required. See the
[private native bridge](../native/electron/public-suffix-display/README.md).

Desktop font enumeration (`fontSettings.getFontList`) remains available. Font
preferences now have installation-owned per-script settings, control metadata,
change events and permission/unload restoration source wiring. They require the
prepared native Session font bridge; the installed binary lacks it, so preference
get/set/clear methods explicitly reject instead of reporting unenforced storage
as success. Existing anonymous flat values are preserved without assigning them
to an extension. Private mutations reject; eligible private inheritance is
resolved separately. Native compilation and actual rendering acceptance remain
required. See [the native font bridge](../native/electron/font-settings/README.md).

Desktop `instanceID.getID`, `getCreationTime` and `deleteID` now use a persistent
local identity owned by the regular profile, with a current `gcm` permission.
Managed Windows page/worker calls, optional-grant revocation, full restart,
disable/re-enable, reset and same-ID uninstall/reinstall cleanup are verified.
Private and sandboxed callers are denied. Token methods and token-refresh events
remain unavailable because the real messaging backend is still missing; native
MV2 and other desktop platforms have not been verified. See the implementation
record for identity storage, worker-lifetime checks and backend revocation gates.

Desktop `declarativeContent` now has persistent, installation-owned rule batches
and browser-owned toolbar effects. Extension pages and workers can construct
`PageStateMatcher`, `ShowAction` and ImageData `SetIcon`, and add/get/remove
`onPageChanged` rules. URL, bookmark and bounded compound-CSS conditions control
the actual toolbar without returning page matches to extension code or granting
page access. CSS observes accessible same-origin documents; shadow-tree/private
pseudo-class and document.domain-relaxed parity remain incomplete. Legacy
`ShowPageAction` now has an MV2 `page_action` owner check and a real toolbar
effect implementation; its live MV2 installation acceptance is still unfinished.
Experimental `RequestContentScript` remains explicitly rejected.
Private execution remains unavailable. See the implementation record for quotas,
native grammar validation, action precedence and remaining acceptance work.

Tab metadata is projected for the current recipient at the main-process return
and delivery boundaries. Without a current tabs permission, matching host grant
or applicable browser-owned grant, `url`, `pendingUrl`, `title` and `favIconUrl`
are omitted. A declared activeTab permission alone is not a temporary grant.
Committed and pending URLs are checked independently. Populated windows,
recently closed/restored sessions and documented browser-event envelopes share
the projection; arbitrary extension messages and script results are untouched.
Nonempty URL/title query filters exclude inaccessible tabs. Title matching uses
Chromium's bounded, case-sensitive Unicode wildcard semantics; empty URL arrays
and empty titles apply no metadata filter. File host access additionally requires
the manager's reviewed file-access grant. Private/session ownership checks still
precede every projection.

On desktop, **Settings → Apps & widgets → Extensions** embeds the full manager
when the bundled Extensions app is installed. A disabled manager remains listed
with an Enable action; uninstalling it removes the subsection. The tab is absent
in private windows. It uses the same install, access-review, enable/disable,
reload, options, and removal flows as the standalone app.

Desktop **Settings → AI**, below Apps & widgets, offers curated installation of
ChatGPT and Claude and a single preferred-extension choice for Summer's AI entry
points. It uses the normal reviewed installation pipeline. General extension
management stays in the Extensions tab. See [preferred AI behavior](retired-custom-ai.md#preferred-ai-extension).

The embedded manager retains an opaque, no-network sandbox. Only its exact frame
can request bounded, fixed manager routes through browser-chrome IPC. The main
process checks the regular browser window and exact enabled bundled package on
each request; the existing CSRF and permission reviews still authorize manager
operations. Ordinary app settings frames do not receive this transport.

The embedded view follows the native Settings header, full-width layout, spacing,
and live theme. Its standalone app badge and compatibility banner are omitted.
A browser-owned Chrome Web Store link opens the fixed extension-catalog URL in
a normal new tab; the opaque manager frame receives no navigation authority.

Desktop extension API preloads contain repeated missing-handler failures. When
Electron reports that `extension-chrome-api:call` has no handler, that preload
rejects further calls locally for one second. One subsequent caller probes the
ordinary authenticated handler; continued missing-handler failures increase the
cooldown up to 30 seconds. Successful dispatch, including a permission rejection
from the handler, restores ordinary calls. Requests are never automatically
retried or replayed, and the cooldown does not grant additional permissions or
disable an extension. This limits IPC and log pressure from a broken runtime;
it cannot unblock a main process already stuck in unrelated work.

Worker handler registration accepts the extension script URL when
Electron has not yet supplied its scope, using the same identity fallback as
request authentication. Ordinary website workers remain excluded. Before a
non-subscription worker API response settles, including a rejection, the main process yields one
event-loop turn while retaining that request's native worker task. This gives
browser work a turn between incoming worker bursts without relying on timers in
Electron's restricted preload realm. The relay retains its 128-call active limit
and FIFO admission; calls and their side effects are never replayed.

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
and `storage.sync` with optional [paired-device Mesh synchronization](extension-mesh-storage-sync.md) are available.
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
fields. Android begins with one normal undiscarded mobile window.
Privileged backgrounds and extension pages also receive
`windows.get`, `getCurrent`, `getLastFocused`, and `getAll`, including
`WINDOW_ID_NONE`, `WINDOW_ID_CURRENT`, optional bounded `populate`, and valid
`windowTypes` filtering. Version 94 adds foreground one-tab normal/popup logical
windows through `windows.create()` and focuses them by activating their existing
full-screen phone tab through `windows.update(id, {focused: true})`. Exact own-
extension URL queries are caller-bound. Android cannot foreground an unfocused
Activity through this extension surface, create floating geometry, or provide
private/inactive/panel/multi-tab windows, removal, or window events. Version 89 also exposes
permission-free `management.getSelf()` in privileged Android QuickJS contexts.
Its required `ExtensionInfo` fields use reverified localized manifest metadata,
authoritative package identity/version, and current API/explicit-host grants;
content-script-only matches are not reported as host permissions. Other
management methods and events remain unavailable.
Backgrounds also receive native `tabs.onCreated`, `onUpdated`, `onActivated`,
and `onRemoved` events with the same URL/title permission filtering. Favicon
metadata is not implemented. Extensions that require `scripting` can execute a
bounded inline function or verified packaged JavaScript files in `ISOLATED` or
`MAIN`, and insert/remove bounded inline or packaged CSS, against the default
main frame, explicit `frameIds`, or `allFrames`. Android/iOS independently
authorize every target frame against effective host access or a same-origin
`activeTab` grant; the grant never substitutes for the separate `scripting`
permission. `documentIds` targeting remains unsupported. Android accepts
`USER` CSS as a documented author-origin WebView fallback with distinct removal
identity; it does not provide Chrome's user-origin cascade or CSP semantics,
and iPhone's custom mobile wrapper still rejects it. Android child-frame IDs are local to each execution world rather than
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
resources. Version 95 accepts a root-relative string or Chrome's size-to-path
dictionary, chooses the entry nearest 32 pixels with the larger size winning a
tie, and freshly revalidates that exact package resource. `ImageData` icons, remaining action
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
compiles and swaps the combined content rule list. Android privileged contexts
can also query and transactionally change up to 5,000 exact-package-validated
disabled static rule IDs with `getDisabledRuleIds()` / `updateStaticRules()`;
this state survives restart and resets on package update, while iOS remains
paused without that surface. Static block rules also preserve
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
Android Version 93 supersedes that early compatibility executor for the active
QuickJS path: explicit `MAIN` remains bridge-free, while a bounded world
configuration can enable authenticated one-shot `runtime.sendMessage()` from an
exact `USER_SCRIPT` world to the background's dedicated
`runtime.onUserScriptMessage` event. Custom world CSP, user-script Ports, and
one-shot `execute()` remain unavailable; iOS work is paused.
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
Electron's existing DNR implementation. The method exists only in privileged
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

Android now connects local signed CRX3 selection to the same verification and
permission-review flow; see the exact
[implementation and device-verification status](android-local-crx-installation.md).
Unpacked installation remains hidden on mobile, and iOS local CRX input is not
implemented. Many browser APIs still have limited or absent mobile support. See
[Mobile WebExtension compatibility — Beta](mobile-web-extensions.md)
for the exact mobile boundary and ranked compatibility targets, and
[Chrome WebExtensions: desktop and phone differences](desktop-mobile-web-extension-differences.md)
for the side-by-side product contract.

## Use the manager

On Electron/Castlabs 44.0–44.3, Summer blocks Chrome Web Store documents in
tabs (including embedded frames) to contain a confirmed browser-process crash
in that runtime. This temporary restriction does not block the Extensions
Manager's signed-CRX download path: paste the Store extension URL or ID into
`sap://extensions-manager/` instead. Remove the restriction only after a
packaged Castlabs build containing Electron's 44.4.0 Web Store fix passes the
disposable-profile Store-navigation regression.

1. Open **Settings > Apps and widgets > Apps**.
2. Open **Extensions Manager**, or navigate directly to
   `sap://extensions-manager/`.
3. Paste a Chrome Web Store detail URL or 32-letter extension ID, choose a
   local `.crx` file, or choose **Load unpacked** and select a directory
   containing `manifest.json`.
4. Review the extension's requested API access, site access, content-script
   matches, optional access, and compatibility diagnostics before confirming.

## Install recommendations

Desktop Summer can offer a browser-owned install notification in an active
regular tab on either:

- any exact Chrome Web Store extension detail URL on
  `chromewebstore.google.com` or the legacy `chrome.google.com` host, after one second; or
- a curated HTTPS website with a publicly documented Chrome extension, after six seconds.

The Store-page case is unavailable on Electron/Castlabs 44.0–44.3 while the
crash-containment guard above is active; installation through Extensions
Manager remains available.

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
manually closing it or choosing **Not now** snoozes extension recommendations for 30
days, while **Don't suggest extensions** disables this category in the local
profile. The policy stores only the category outcome and timestamps, never the
visited URL, site name, or extension ID.

Notifications remain visible without an automatic timeout while their page is
active. Starting a top-level navigation, leaving the page within a single-page
site, switching tabs, or closing the tab dismisses them promptly. Automatic
dismissal releases the reservation without adding a 30-day snooze. Web Store
visits bypass only the recent-typing suppression so typing the address does not
silence the one-second prompt; other install-tip policy gates still apply.

Visiting an installed extension's Web Store detail page instead shows an
**already installed** notification with **Manage extension**, **Options** (only
when the extension is active and provides an options page), and **Remove
extension…**. Management opens that extension's record, including disabled or failed
extensions; removal opens the existing manager confirmation without uninstalling
anything automatically. These contextual controls do not use the install-tip
budget, snooze, or opt-out. They appear once per tab visit after one second,
only in an active regular tab. Navigation, tab closure, or switching away closes
the notice, and actions recheck the current tab, window, and installed record.

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

Summer attaches service-worker lifecycle observers before its initial running
worker scan. At each starting or running generation it reclaims only Summer's
worker API channel by removing any stale handler for that channel and installing
the current handler. Electron normally retains the handler across an idle stop;
the bounded rebind also repairs an empirically observed missing registry without
stacking duplicate handlers or changing another extension IPC channel.

Managed extensions can remove themselves with `management.uninstallSelf()`
without requesting the broader `management` permission. Summer authenticates
the calling extension and removes only its own manager record, regular and
private runtimes, grants, and browser-owned package data. The default silent
form and `{showConfirmDialog: false}` are supported; Summer rejects
`{showConfirmDialog: true}` until it has a browser-owned confirmation surface.
Extensions with the effective `management` permission can call
`management.uninstall(id)` for a Summer-managed extension or website app in
the regular profile. Removing another installation always requires confirmation in browser
chrome, even if the request sets `showConfirmDialog: false`. Summer revalidates
the initiating extension and the exact target installation before removal.

`management.generateAppForLink` installs a profile-owned HTTP(S) website app
with browser confirmation. `get` and `getAll` expose these items as hosted
apps. `launchApp` opens the saved URL in a regular tab or a normal Summer
window according to the saved mode. `setLaunchType` confirms changes to that
mode. `createAppShortcut` confirms Windows desktop shortcut creation and
binds the link to the profile and exact app installation, so it uses the current
saved mode and cannot be reused after reinstalling the app.
The four website app methods require a visible regular-profile extension
page. Users can also manage website apps from Summer’s Apps panel.

The matching `runtime.setUninstallURL()` registration accepts an empty string
or a bounded HTTP(S) URL, persists it with the managed extension, and opens it
only after removal commits. Summer does not open uninstall pages during reload,
disable, failed removal, or browser shutdown.

Compatibility diagnostics are derived from the manifest before extension code
runs. Summer allows Electron-supported extension APIs such as `runtime`,
`storage.local`, `tabs`, `scripting`, `webRequest`, `webRequestBlocking`,
`management`, and DevTools extension pages. Summer also adds browser-owned
compatibility shims for extension pages and Manifest V3 service workers through
dedicated `chrome-extension://` frame and service-worker preloads.

On Electron 45.1+ (including the 44.10 backport used by the macOS packages),
native session preload URL filters keep extension augmentation in extension
documents and worker scopes. The website preload excludes extension documents
and DevTools, with a separate path-filtered registration for Claude's existing
boolean-only panel authorization bridge; its exact URL validation remains in
the preload. Browser-chrome IPC preloads are limited to the production chrome
URL and the configured development chrome URL. Origin fallback is explicitly
disabled so originless children do not inherit extension or browser-chrome
bridges. Website originless frames retain their existing behavior, and older
development engines retain the same preload-side guards. Main-process sender
authorization remains mandatory for every exposed operation.

Desktop ISOLATED content scripts receive a storage-only bootstrap before their
own JavaScript, using their existing match, timing and frame declarations. The
bootstrap connects to a separate browser-owned native broker, so `storage.sync`
shares the page/worker provider even in MV2/MV3 extensions without a background.
It exposes no Electron or website bridge and adds no API to MAIN-world scripts.
See [content storage](extension-content-storage.md) for native identity checks,
private-session restrictions and immutable snapshot migration.

Manifest V2 backgrounds require one versioned browser-owned context-menu bootstrap because
Electron does not apply those preloads to `backgroundPage` WebContents. Summer
adds that bootstrap only to the temporary browser-owned staging copy before its
final content hash and access review; loading an approved snapshot is read-only.
Each preload or staged bootstrap exposes a narrowly scoped broker, then installs
the compatibility surface in the extension main world before its own scripts
execute.
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

The regular top-level website messaging bridge distinguishes a failed
connection from an actual empty reply. Missing or inaccessible extension targets
receive the same generic connection failure, as do unavailable website receivers
and pending website deliveries lost to timeout or extension teardown. Promise
calls reject; callback calls expose `runtime.lastError` only during that
callback. An actual receiver response of `undefined` or `null` remains valid.
This allows websites to try another extension build after a failed connection
without disclosing whether the first target was installed but inaccessible.
Manifest, current-frame and regular-profile authorization remain unchanged.

A direct HTTPS child of a visible, browser-owned extension side panel may use
that same narrow messaging method to contact only its owning extension. The
browser authenticates the native parent, current panel declaration, child
frame and presentation lifetime on every delivery; manifest origin matching
still applies. This auxiliary sender has no `sender.tab`, matching Chrome's
panel context. Ordinary website iframes, packaged tabs/popups, hidden panels,
nested descendants and private profiles receive no such authority. Navigation,
replacement, tab activation or closing revokes an in-flight delivery.

The service-worker preload instead exposes one fixed, call-only `contextBridge`
capability. Summer authenticates its `ServiceWorkerMain` sender and bounds each
request and response before the main-process broker handles it. Validated
main-to-worker events use one preload-owned callback captured during worker
bootstrap, before extension code can scuttle ambient globals. The callback is
released with worker teardown. No Electron, Node, filesystem, or raw IPC object
crosses into extension code. Worker permission and command reads remain derived
from the reviewed manifest. The relay runs at most 128 authenticated API calls
concurrently, while a FIFO can retain up to 2,048 calls and 64 MiB of serialized
request data. This accommodates extensions that probe large static rulesets in
parallel without allowing their burst to become unbounded.
The worker's `tabGroups` overlay also supplies the same frozen `Color` enum and
`TAB_GROUP_ID_NONE` constant as extension pages, synchronously before MV3 module
imports evaluate. These public constants do not call the broker or add
permissions. Incomplete or frozen native namespaces remain unchanged; methods
and events continue through the authenticated worker relay.
Those shims expose brokered real-profile cookies, bookmarks, browsing history,
download records, alarms, notifications, debugger targets, popup windows,
proxy settings, privacy/content-setting preferences, commands, context menus,
side panels, font settings, permissions, supported browsing-data removal, and
web navigation. The machine-readable registry in
`shared/chromeExtensionCompatibility.ts` is authoritative for each member.
For `tabs.create()` and `tabs.update()`, Summer resolves relative destinations
against the authenticated calling extension page or worker script before the
browser host creates or navigates the tab, matching Chrome's extension URL
semantics without trusting a renderer-supplied extension identity.
`tabs.goBack()` and `tabs.goForward()` use Summer's browser-owned history for
the exact tab ID returned to the extension. They accept an omitted ID for the
active tab in the current profile window and reject stale targets, popup tabs,
and history involving internal pages outside HTTP(S), exact `about:blank`, or
the calling extension's own pages. They preserve the normal navigation warning
and private-profile ownership checks.
`tabs.duplicate(tabId)` uses native saved history to copy form/scroll state,
session storage and zoom. A normal copy is active and adjacent to its source,
with group membership prepared before creation events; the native Duplicate Tab
menu creates a background copy. An owned extension-popup source produces a new
popup with the source's Session, restored size and extension ownership, including
when another extension requests the copy. Copies remain usable after their
source closes. Returned metadata uses the caller's current grants; duplication
does not create an `activeTab` grant. Windows regular-MV3 pages and workers are
verified through both aliases, Promise/callback and current/held bindings.
Saved native state can lag the latest renderer changes. Full preference copying,
split/pinned parity, positive private execution, MV2 and other operating systems
remain unverified; this is partial Tabs support. See the implementation record
for the accepted evidence and preserved failed diagnostics.
`tabs.captureVisibleTab()` returns a bounded JPEG or PNG data URL for the active
regular HTTP(S) browser tab in the selected Summer window. It accepts Chrome's
optional window ID and image options overloads, and permits at most two captures
per second per extension. Summer requires the extension's reviewed `<all_urls>`
host permission or a current browser-owned `activeTab` grant for that exact tab
and HTTP(S) origin. Only an authenticated toolbar invocation creates that
temporary grant; declaring permission or calling action show/enable does not.
The caller's installation, document/worker lifetime and current permission are
rechecked after capture, alongside the browser's tab/navigation checks.
Private windows, internal pages, extension popups, and changed or stale active
tabs are rejected.
MV3 worker calls to `chrome.debugger.sendCommand` can receive a
`Page.captureScreenshot` result up to 12 MiB through the authenticated relay;
other debugger results keep the 1 MiB generic response limit.
`windows.create({focused: false, url: "about:blank"})` creates an ordinary
regular-profile Summer window with one tab through the normal browser-window
lifecycle and leaves the currently focused window active. Explicit
`type: "normal"` also accepts one HTTP(S) URL. Extension-owned pages still use the
separate popup path. The creating extension can later set that regular window's
width, height, and `state: "normal"`; other extensions cannot resize it.
Positioning and private-window creation remain unavailable through this API.
The four window read methods validate `populate` and `windowTypes` and filter
the process's registered owners. Omitted, null and empty type lists
use the normal/popup default; valid types with no registered owner yield no
matches. An explicitly associated caller's `getCurrent()` and `get(-2)` retain
Chrome's current-window fast path before type filtering. Explicit `get()` IDs
must be integers from -2 through 2147483647; numeric strings are rejected.
`getLastFocused()` filters before selecting from native focus observations,
which begin when the browser registers each window. Earlier focus history is
unknown and retains a deterministic fallback. Populated window queries select
their tab owners before reading unrelated tab/window metadata. A disposable
Windows regular-profile MV3 browser test passes 588 public calls, including
585 reads through pages/workers, both aliases, Promise/callback forms and held
references across a real second extension install. It checks populated native
tab/window identities, observed native focus order, filtered and invalid-query
errors, and exclusion of a real private window without an extension grant.
Page window callbacks use the shared page-owned error adapter so `lastError`
is visible during the callback and restored afterward. Earlier failed browser
and cleanup evidence is retained. Private-positive, MV2 and other-platform
behavior remain unverified. Native synchronous argument-error timing is not
established by this implementation.
Query type names do not enable creation of unsupported window types.

With the native managed-DevTools owner ABI, Main enrolls the actual detached
`BaseWindow` as `devtools` in the existing window table. Explicit window queries
can select it; default queries still select normal/popup windows. Docked tools
use the inspected window's current owner without changing its type. Retired
detached owners disappear from reads and mutations immediately, and emit removal
once their actual native window closes. Stock Electron without this ABI admits
no DevTools owner.

Populated detached windows expose their actual frontend member with public tab
ID `-1`, index `0`, and the actual owner window ID, following the selected Chrome
serialization. A dedicated `tabs.get(rawNativeId)` read and detached frontend
`tabs.getCurrent()` use that projection. They do not grant ordinary tab mutation,
messaging, capture, or `tabs.query()` membership. Docked `tabs.getCurrent()` returns
undefined: initial and redocked Chrome SessionTabHelper history is not modeled
without a native witness or differential browser evidence. Runtime context IDs
are a separate native contract.

Numeric DevTools window updates validate all supported geometry, state, focus,
and attention fields before native mutations, including display intersection and
state combinations, and recheck owner lifetime after native calls. Numeric
DevTools removal is rejected; `windows.remove(-2)` can close the real frontend's
associated owner. Another page or worker has no such association. ChromeOS
locked-fullscreen, native DevTools/browser execution, other-platform behavior,
and complete Chrome parity remain unverified by these Main unit regressions.
An MV3 worker can group that new window's tab with `tabs.group()`, label the
group through `tabGroups.update()`, and later recover its members with
`tabGroups.query({title})` followed by `tabs.query({groupId})`. A disposable
extension test covers this sequence after the worker discards its in-memory
group ID; it does not simulate Chromium service-worker eviction or a browser
restart.
Extensions Manager shows each installed extension's ID; use that exact ID in
the native messaging host manifest's `allowed_origins` entry.
On macOS, discovery includes stable-channel Edge hosts installed for all users
at `/Library/Microsoft/Edge/NativeMessagingHosts/<host-name>.json`, after the
existing per-user and system locations. The same manifest, origin, and executable
validation applies to every location.
`sendNativeMessage()` reports broker failures through a rejected Promise or
callback-scoped `runtime.lastError` in the extension page's own JavaScript world.
The callback form returns synchronously, and restores the prior error descriptor
when the callback returns or throws.
Native messaging accepts up to 64 MiB of JSON payload from an extension to its
host, including long recording traces through the MV3 worker bridge. Each
long-lived Port can queue at most one maximum-size framed message, and all
native-host Ports together can queue
at most two; a host that stops accepting writes is disconnected. Messages from
the host to the extension remain limited to 1 MiB. Windows host launch resolves
only the recognized Node batch wrapper without running a shell.

Desktop `windows.create` uses Summer's ordinary browser-window factory for
regular windows, with at most 32 HTTP(S), `about:blank`, or same-extension URLs.
It can move an existing regular tab into a new regular window without replacing
that tab's renderer. Popups support one allowed page; popup tab transfer, private
window creation, and opener inheritance remain unavailable. Bounds/state changes
and removal require the calling extension to own the created window; unrelated
browser windows retain focus-only mutation. Inactive creation selects a tab only
within its window and preserves the global browser-command owner. The shared
`front/tabWindowActivation.ts` helper keeps those two selection states distinct.
These source paths have focused coverage; see the live verification record for
the current acceptance status.
`tabs.getCurrent()` returns the authenticated calling tab for normal Summer
tabs and extension-created popup windows, and returns no tab for service
workers or browser-owned action popups. Extension-page favicons are copied from
the same loaded extension package into a bounded raster data URL before the
file-backed browser chrome renders them; raw cross-origin extension resource
URLs never cross into the toolbar renderer.
Extension pages and workers with effective `favicon` permission can fetch their
own `chrome-extension://<id>/_favicon/?pageUrl=...&size=...` endpoint. This bounded
fetch bridge returns PNG bytes from the exact current regular tab's cached
raster; it performs no network lookup. Allowed sizes are 16, 32, 64, 128, and 256.
Native image conversion falls back to a fixed decoder in browser-owned isolated
world 954, with document/cache checks before and after execution. Cancellation
and a five-second delivery deadline retain decoder admission quota until the
underlying execution settles (four per caller, eight per extension, 32 total).
Private tabs, unopened/stale URLs, missing grants, and invalid images are denied.
Each live tab retains at most four validated cached rasters for exact HTTP(S)
document URLs. When Chromium reuses an unchanged icon without another favicon
event, restoration waits for that exact replacement document to commit and stop
loading. New favicon updates invalidate the older entry, and tab disposal clears
the history; navigation never grants access to a URL that is no longer current.
This does not implement reserved-resource image URLs or complete native Response
metadata: bridge Responses have an empty URL and default type. See the
[live verification record](ai-extension-integration-tests.md) for current evidence.
Required Chrome API permissions are warnings rather than install blockers so
real extensions can load and reveal the next compatibility gap at runtime.
Malformed manifests and changed permission fingerprints are still rejected or
sent back through review before code runs.

Chromium refreshes existing page namespace properties when another extension
loads or unloads. Summer preserves its composed methods and event objects on
Chromium's cached native namespaces after capturing native delegates; final page
runtime and scripting wrappers are published after preload setup completes.
Native namespace availability and permission revocation stay Chromium-owned.
Callback-scoped errors restore the original native error descriptor, and this
path neither polls for resets nor reinstalls listeners. The integration record
includes a live second-extension load/unload regression.

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

Installed extensions disabled in Extensions Manager remain visible in regular
windows as grey, manage-only toolbar badges, including in the overflow menu.
Because their runtime and declared icon are unloaded, the badge uses the
extension's initial rather than executing or reading an inactive action.
Selecting one opens its exact manager record after the browser rechecks that
the installation is still disabled. Private windows do not list disabled
regular-profile extensions.

Summer provides browser-owned `chrome.action` and `chrome.browserAction`
state, including titles, icons, popups, badges, enabled state, and per-tab
overrides. Extension pages and Manifest V3 workers mutate that state through
the authenticated broker, and the toolbar snapshot updates from the same
authoritative store. Click-only actions without a declared popup wake their
managed worker when necessary and dispatch `onClicked` with the active Summer
tab.

MV3 `action.getBadgeTextColor()` and `action.setBadgeTextColor()` read and update
the same default/per-tab badge state used by both the toolbar and its overflow
menu. An unset getter returns `[0, 0, 0, 0]`; rendering chooses contrasting text
until an explicit color is set. Colors accept four integer RGBA channels or
Chromium's legacy named/hex/rgb/hsl CSS syntax, with CSS strings limited to 512
characters. A fully transparent text color rejects without changing state.
These two methods belong to `action`, rather than the MV2 action namespaces.
Real page/worker callbacks, inheritance, rejection, narrow-window styling and
English/Hebrew layout have passed the focused browser fixture. This does not
establish parity for the other missing action methods.

MV3 `action.openPopup(options?)` uses the same sandboxed browser-owned popup
view as a toolbar click. It accepts an optional signed-32-bit `windowId` (or
`windows.WINDOW_ID_CURRENT`), requires an active accessible window and an
enabled action with a popup, and rejects if another action popup is open.
Programmatic opening does not grant `activeTab`, dispatch `onClicked`, or
trigger the side-panel click override. Promise completion returns `undefined`;
callbacks receive no arguments and observe errors through their own
`runtime.lastError`. The initiating installation, caller, profile, active tab
and URL are rechecked after loading. Closure during loading rejects; a
10-second limit closes a stalled view. API popups anchor at the browser's
action area, including overflow actions. The parent integration gate passed
623 tests across 21 suites. A real foreground Windows fixture verified attached
popup DOM for a page Promise and worker callback, repeat-call rejection,
browser-owned dismissal, invalid-window/disabled/popup-less errors, scoped
callback errors, and no URL-metadata/capture authority or click event after
programmatic opening. The existing MV2 page-action browser regression also
passed. The new fixture checks an attached browser view; URL presence alone can
include a helper page and is not acceptance evidence. Earlier loopback-navigation
and URL-only dismissal fixture failures remain preserved. Evidence is
`.tmp/action-open-popup-live-evidence-parent-20261027.json` and
`.tmp/action-open-popup-live-parent-r5-20261027.log`. This method is not
added to MV2 `browserAction` or `pageAction`.

MV3 `action.getUserSettings()` and `action.onUserSettingsChanged` now have a
browser-owned pin-state implementation under focused verification. The getter
returns the committed `{isOnToolbar}` preference, independent of window width or
temporary overflow. An authenticated native action context menu owns the Pin to
toolbar checkbox; extensions receive no setter. The manager persists before
committing state and emits only actual changes. Ordered delivery targets that
extension's current installation in the regular Session, with a fresh ownership
check across worker wake. No `activeTab`, action-click event, side-panel override
or user gesture is created. A failed save, unchanged value, resize, reload or
restart emits no settings change.

Pins are an additive optional field in the existing format7 installation record.
Older records without it retain their previous inline eligibility; newly accepted
installations start unpinned. Update/reload/disable preserve that record's pin;
uninstall removes it, so a new installation starts unpinned. Private windows read
the regular preference only for their explicitly allowed active runtime and cannot
mutate it. The pinned Chromium context-menu caller also forbids pin changes in
private windows (`extension_context_menu_model.cc:482-488`); its dispatcher
restricts the settings event to the originating regular context and marks no
gesture. Split private workers do not receive a manufactured duplicate event.
The final coordinated Windows fixture passes in English and Hebrew, including
page/worker settings and events, extension isolation, unchanged-value/resize
behavior, restart persistence and private read-only inheritance without duplicate
private events. All eight related browser regressions pass. Evidence is
`.tmp/action-pin-integrated-live-evidence-parent-20261027.json`. The owned fixture
drives the real keyboard route and native MenuItem callback; OS menu presentation
and hit-testing are replaced and remain outside this acceptance claim.

The reusable `extensionActionUserSettings.ts` sink serializes each extension's
committed changes across asynchronous worker startup, retains only the current
per-extension delivery chain, and drops obsolete installation identities before
or during recipient delivery. It has no UI, storage or context-discovery authority.

MV2 `page_action` declarations use `chrome.pageAction` with `getPopup`,
`getTitle`, `hide`, `setIcon`, `setPopup`, `setTitle`, `show` and `onClicked`.
The toolbar keeps an unshown action grey; manual per-tab state resets on a
committed cross-document navigation, while same-document navigation preserves
it. Declarative `ShowPageAction` requires that actual manifest owner, and an
explicit hide takes precedence over its rule. Native MV2 backgrounds receive
the action methods/events through the same authenticated hidden relay used by
the existing background bridge. Its JSON transport snapshots known own details
once, preserves hidden tab IDs, and copies bounded icon pixels before delivery.
Both Chrome and a distinct browser alias receive the owned action event.

The reusable frame Port installer captures the original native `connect`
delegate, receiver and manifest reader before cached-namespace publication can
replace them. It retains them only in its closure after the temporary capture
state is deleted. Native routing requires the same captured top-level document,
exact extension identity and reserved MV2 relay HTML address, a current MV2
background manifest, and a single own-data `{name}` argument naming `:api`,
`:web-navigation` or `:runtime-port` on `summer-mv2-context-menu-v1`. The generated
relay uses the current guarded `runtime.connect`, never its mutable prototype.
Other documents, external targets and malformed or ordinary Port calls retain
the authenticated Summer broker. This restores the relay's intended native hop;
it exposes no native delegate or general native-Port capability to extension code.

A trusted toolbar invocation can grant temporary HTTP(S) tab metadata and
visible capture to the current reviewed `activeTab` installation. Same-origin
reload/navigation retains it; cross-origin navigation revokes it permanently,
including after history returns. Showing/enabling an action creates no grant.
Real MV2 background delivery, capture, tab isolation, navigation revocation and
popup acceptance pass in the disposable browser fixture. Native scripting/CSS
permission grants, other invocation sources and special-URL parity remain
incomplete; the portable grant is not native Chrome permission parity.

The MV2 page-action fixture has a bounded, fixed background-stage diagnostic for
platform failures. It checks the exact same-session native background main frame,
reads listener/action-entry and native-write start/return/throw counts, then reads
only its synthetic storage key for equality. The original native write remains
fire-and-forget: return does not prove settlement. A separate readback can prove
the key became visible without observing its Promise or consuming rejections.
Native and controller readbacks stay separate; diagnostic failure cannot delay
strict cleanup beyond three seconds or replace the original assertions.

At the fixture background's synchronous startup, a passive native-Port observer
checks only the exact browser-owned API relay name and native sender ID/URL,
then counts connections, disconnects, generic API response success/rejection and
matching `pageAction.onClicked` envelopes. It never wraps a Port method, sends a
message, registers an API subscription or reads event payloads. Response envelopes
contain no API/event identity, so these counts are not page-action subscription
ACKs; outgoing queue/request state and an earlier missed connection remain unknown.
A separate unique same-session, installed-path/version and stable-root-frame
check reads only the hidden relay's existing `hasListeners` and readiness-function
presence. It never invokes private readiness with a fabricated listener. Only
fixed counts/booleans leave either receiver; all original assertions remain.

The toolbar's main-process integration can use the prepared private native
Session grant/revoke pair when both methods are available. It waits for an exact
renderer-acknowledged tab/origin/installation receipt before action delivery and
portable disclosure, then rechecks the installation, active tab, URL, focus and
action state. Stale owners are revoked with their original activation epoch,
including after reentrant or late native completion. Internal/file/opaque pages
can still deliver ordinary actions without granting host authority. Private
sessions keep the portable metadata path and never call that native bridge.
The installed binary lacks the pair; its portable behavior is not evidence that
native scripting or stylesheet permission grants work.
The current real Windows fixture verifies focused HTTP(S) actions and ordinary
about:blank click/popup delivery with URL/capture access still denied there.
Because trusted toolbar activation rechecks native focus, that fixture requires
runner-level visible-window consent on Windows rather than hidden locator input.

On ordinary page right-clicks, Summer composes matching extension-supplied
`contextMenus` commands into its native browser menu. A single visible normal
root is shown directly; multiple roots expand beneath the extension's name,
matching Chrome's grouping behavior. Manifest V2 background pages and Manifest
V3 workers both use the same browser-owned persistent menu store, so a
provisional Electron namespace cannot register commands that the visible Summer
menu never receives. The `menus` permission and namespace are authorized as the
Chrome-compatible `contextMenus` alias. A suspended Manifest V2 event page is
woken through its same-extension relay; Summer holds the click until the
extension's background script registers its listener, and only then acknowledges
delivery. Manifest V3 workers may immediately message the clicked page from
`onClicked` or from the content-script message that records the right-clicked
element; Summer lets either extension event dispatch unwind before entering
Electron's native `tabs.sendMessage` binding, matching Chrome's queued delivery
without terminating the extension host. Before entering that native boundary,
Summer resolves the exact registered target and verifies its public tab ID
equals the actual `WebContents` ID, so native and broker delivery address the
same contents.
Manifest V2 listeners that existed before a programmatic script injection stay
on Electron's native event, while listeners registered by injected code stay on
Summer's isolated-world bridge. A tab message dispatches to both disjoint sets
and returns one response without registering either listener twice. Background
API and web-navigation requests waiting for their hidden relay are capped at 128
per relay and expire after ten seconds; an expired mutation is removed instead
of being replayed if the relay later reconnects.
Every asynchronous Chrome API request issued by a Manifest V3 worker also owns
a native worker task until the request settles. This includes popup creation:
the worker cannot suspend after showing an extension window but before storing
its state and sending the page's initialization message. A
trusted right-click hit-test also
preserves an underlying image, video, or audio context when a site places a
similarly sized clickable overlay in the same media container; unrelated overlay
trees remain ordinary page or link targets.

Right-clicking an extension action opens its matching extension-supplied
`contextMenus` tree followed by browser-owned controls. Summer preserves
`parentId` hierarchy as native expanding submenus and applies Chrome's six-item
top-level action-menu limit without truncating submenu children. **Options** appears
only when the active managed extension declares an options page; **Manage
extension** opens and focuses its exact Extensions Manager record; and **Remove
extension…** opens that record's existing confirmation dialog. Summer
revalidates the active tab, action, and managed runtime mapping before showing
the menu and again before opening a browser-owned destination. Browser-owned
menu definitions survive disable/re-enable and runtime reload, but a committed
uninstall purges them so reinstalling the same extension ID cannot revive stale
menu state.

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
rule. Summer reads, validates, and compiles rule updates outside the intercepted
request path, constructs separate immutable static and mutable matcher indexes,
and publishes their combined generation with one main-process assignment before
the update API completes. A dynamic or session-rule update rebuilds only the
mutable index; large installed static blocker lists remain untouched.
`onBeforeRequest`, request-header, and response-header decisions then run
synchronously against that snapshot and settle Electron's native callback in
the listener's original stack. This mirrors Manifest V3 DNR's browser-owned
decision model and avoids Electron 43's phase-unsafe asynchronous response
storage. Observer-only `webRequest` events are delivered without delaying the
native callback. Their retained registrations wake an idle Manifest V3 worker
and deliver the event after that native callback has already settled. A genuine
blocking listener retains Summer's authenticated, bounded asynchronous response
path. Listener errors fail open.

The worker matcher remains available as a benchmarked future option, but it is
not used by the native interception path while Electron stores asynchronous
blocking phases in one request-ID-keyed slot. Re-enabling worker-side request
matching requires the phase-safe Electron fix and a new live stress pass.

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

Mutable stores currently belong to each native Session and clear when the
extension unloads, including disable/reload. Dynamic rules also lack durable
browser-restart and extension-update persistence. Chrome's
[dynamic rules persist across browser sessions and upgrades](https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest#dynamic-and-session-scoped-rulesets).
Session rules are correctly memory-only, but Summer's every-unload reset and
separate regular/private mutable stores remain compatibility differences.
These four methods are therefore partial support; the accepted static-settings
and cache-owner changes do not alter their existing lifetime or privacy scope.

Desktop `getDisabledRuleIds({rulesetId})` and `updateStaticRules({rulesetId,
disableRuleIds?, enableRuleIds?})` use that same authenticated main-process
matcher in extension pages and workers. IDs are signed 32-bit integers; unknown
IDs are retained, duplicates collapse, and disable wins a collision within an
individual-rule update. The per-extension limit is 5,000 disabled IDs across all
declared static rulesets, including currently disabled rulesets. Disabled IDs do
not release static or regex allocation. Whole-ruleset updates still use
enable-wins semantics and retain individual masks.

`ExtensionDnrStaticRules` stores enabled-ruleset selection and masks under the
manager's record identity and deterministic installation revision. Restart and
disable/re-enable preserve them; packed updates and explicit unpacked reload
reset them. Legacy installations start with manifest defaults and no disabled
IDs. Dynamic and session-rule lifetime behavior is unchanged. A serialized
operation validates every ID and quota, prepares all eligible loaded matchers,
saves synchronously, and then publishes complete generations. Failed saves or
stale callers publish no candidate generation. Regular and explicitly permitted
private runtimes share these static settings, matching Chromium's redirected
service; existing session, permission and sandbox checks still apply. No worker
matcher is published during preparation: network evaluation uses immutable,
synchronously selected main-process generations. The reference is Chrome's
[static-rule API](https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest#method-updateStaticRules)
and pinned Chromium `prefs_helper.cc` (disable wins and update lifetime),
`declarative_net_request.webidl` (signed `long` IDs), and
`rules_monitor_service.h` (incognito redirection).

Page and worker bootstrap use the same fixed twelve-method Summer DNR owner.
When the native per-context cached namespace is replaceable, both aliases are
preflighted before its methods are changed, so Chromium binding refresh returns
Summer's methods rather than a separate native rule store. Method descriptors
are verified after all root/namespace publication, and failed installation
verifies restoration of the attempted changes. The page's specialized frozen
view retains the cached namespace as its prototype after copying its own fields.
An explicit reinstall updates the cache atomically; held functions continue to
use current main-process authorization and the actual public runtime error scope.
The twelve methods include the four former page helper fallbacks, preserving
Summer's quota, regex, feedback and action-option backend instead of accidentally
selecting native helpers. Existing native constants and events are retained.

Windows managed-MV3 tests verify real regular and explicitly permitted private
filtering across two extensions, current/held methods through both aliases,
static restart/reload behavior and private revocation. The focused integration
gate passes 678 tests. Management and English/Hebrew side-panel regressions pass
on the same built code. The initial private fixture selected a stale manager
page; its failed campaign remains recorded, and a real Refresh correction passes
the private case separately. Evidence is
`.tmp/dnr-full12-acceptance-parent-20261028.json`.

Missing/delayed native caches, immutable or incompatible descriptors and fixed
non-enumerable page methods use a bounded overlay when it can be published;
otherwise installation reports failure. Refresh continuity is not accepted for
these shapes. Other platforms and MV2 are not live verified. This does not establish
full DNR parity, additional feedback behavior or mutable-rule browser-restart
persistence; existing main permission and private-session boundaries remain.

Redirects accept one validated direct URL, extension path, regex substitution,
or complete URL transform. Extension-controlled `urlFilter` and `regexFilter`
matching, `regexSubstitution`, and `isRegexSupported` operations compile through
the shared `re2js` adapter instead of JavaScript `RegExp`. DNR patterns are
ASCII-only; rules default to case-insensitive matching, while
`isRegexSupported` defaults to case-sensitive matching and honors
`requireCapturing`. Syntax failures and the Chrome-shaped 2 KiB compiled-program
cap report `syntaxError` or `memoryLimitExceeded` without executing an unsafe
pattern. Generic relay calls preserve omitted flags and pass malformed flag
values to the same validator as dedicated DNR calls; they do not coerce invalid
values into valid booleans. Included response-header conditions use any-match
semantics, excluded
conditions take precedence, and late block or redirect actions run only after
response headers exist. Summer-owned DNR methods in extension pages and workers
use authenticated dedicated bridges or the generic relay. With
explicit `declarativeNetRequestFeedback` permission, `getMatchedRules`
returns bounded recent-match telemetry under a 20-call-per-10-minute quota.
Summer does not substitute a declared `activeTab` permission for a browser-owned
grant, and calls are not gesture-exempt until the relay can authenticate a user
gesture. `setExtensionActionOptions` validates live tab ownership atomically and
connects non-allow rule counts to the extension action badge. Rules outside
Summer's supported condition subset are skipped rather than partially applied.
Imported or custom uBO Lite filters that rely on unsupported DNR conditions are
not a complete parity target.

Chrome's `webbundle` and `webtransport` resource-type values are retained when
rules are compiled. This prevents a rule that also names an Electron-observable
type from being discarded. Electron does not currently classify requests as
either type, so a rule restricted exclusively to one of them cannot match until
Electron exposes that distinction.

Summer's main process also observes request lifecycle events and forwards
Chrome-shaped `webRequest` details only to active extensions that declared the
permission and match both the request URL and its initiator through their host
permissions. Each listener's URL, resource-type, tab, and window filters are
enforced before delivery. Request headers, response headers, and upload data
are included only when that listener requested the corresponding
`extraInfoSpec`; listener removal also removes the browser-side subscription.
Blocking responses are resolved within Summer's bounded 500 ms deadline.

Windows regular-MV3 browser acceptance covers tab highlighting. `tabs.highlight` takes strip
indices, activates the first requested index and returns a populated Window with
each tab's URL/title fields projected using the caller's current grants. Summer's
main process owns the highlighted set and range anchor separately from the
active tab. Queries, `tabs.update({highlighted})`, close/reorder/move operations
and the trusted tab strip use that same state. Revisioned renderer snapshots
acknowledge native activation before updating the legacy scalar, preventing an
API selection from being echoed back as a one-tab replacement.

`tabs.onHighlighted` reports complete ID sets without exposing internal native
provenance. It also observes meaningful active/anchor changes. Delivery stays in
the actual source profile and rechecks installation, source and listener lifetime
when a retained worker wakes. Per-listener ordering includes the transition from
suspended to live; unsubscribe or reinstall retires queued facts. Page/worker
Tabs methods and events use fixed-schema native-cache transactions, public-realm
callbacks and rollback. Repeated setup carries bounded worker listener state only
when its privileged relay is unchanged; this metadata grants no authority.
The tab strip sends selection on primary pointerdown because its drag handler
cancels compatibility mouse events. The real pointer, modifier, keyboard,
narrow-window, selected-close and popup paths pass along with exact no-op and
fresh-event checks. Two current/held page/worker matrices cover `highlight`, `get`
and filtered `query`, both aliases and Promise/callback forms across a real
second extension load. Actual native worker stop and event-driven restart pass;
ungranted private selections and mutations stay isolated. Separate ordinary-close
and Hebrew-layout regressions pass on the same build. This does not verify
private positive access, MV2, other platforms, split-tab parity or modified/delayed
native cache shapes, and does not claim all forced Tabs methods were browser-tested.
Evidence: `.tmp/tab-highlight-acceptance-parent-20261028.json`.

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
`onCreatedNavigationTarget` has Windows regular-MV3 page/worker browser acceptance
for genuine native window-open targets. Independent source review, the native
creation probe and same-build lifecycle/selection regressions pass. It captures the exact
native source frame before adopting the actual guest, joins that guest to a
published browser tab or website popup, and publishes the six Chrome event fields
before its held navigation facts. An absent native opener can resolve through the
actual navigation initiator; unknown frames never fall back to the main frame.
The native guest keeps Chromium's initial navigation, including request bodies
and referrers. Named-target reuse is not a creation. Pending creations and held
facts have bounded capacity and lifetime. Event audiences are captured at native
observation; later listeners cannot inherit old facts. Private sessions remain
excluded. The accepted fifteen-target case also proves exact public/native target
ownership, original form POST preservation, same-realm permission grant/revoke/
regrant, revoked listener cleanup, no historical replay and real stopped-worker
wake ordering. It counts as partial support; MV2, other operating systems,
allowed-private observers, immutable/delayed cache shapes, source-frame-removal
parity and full namespace behavior remain unverified. Acceptance and authoritative
retained raw evidence are pinned in
`.tmp/webnavigation-created-target-acceptance-parent-20261028.json`.
Prerender-specific `onTabReplaced` remains unavailable. MV3 worker `webNavigation` subscriptions
are retained by Summer's bounded service-worker relay so registered navigation
events can wake a suspended worker when Electron omits that namespace.
Navigation capture and frame queries admit native regular-profile tabs only;
private sessions have no `webNavigation` authority. Internal source registration,
current recipient permission and retained listener lifetime are checked again
when queued events wake a worker. Automatic worker re-subscription retains that
listener lifetime and refreshes its validated URL filter; unsubscribe and unload
retire it. A Windows managed-MV3 regression proves regular page/worker listeners
and frame queries exclude actual private main-frame and iframe navigations,
while regular navigation still arrives before and after them. The earlier build
reproduces the private metadata leak. This does not add private API support.

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

### Website content settings

Summer implements extension-owned `contentSettings` rules for `location`,
`notifications`, `camera`, `microphone`, `clipboard`, and `sound` using its existing Electron
runtime. Permission rules govern native checks and requests and the website
camera/microphone status bridge; sound rules govern native tab and popup mute.
User permission Deny remains authoritative. Sound blocking composes with user
mute: removing an extension rule does not unmute a tab the user muted.
Permission rules apply to subsequent native checks and requests. Setting Block
does not revoke a media stream already granted to a document, end an existing
geolocation watch, or provide Chrome permission-change subscription events.
Previously granted capture ends through its normal stop/navigation lifecycle.

Clipboard rules govern the original native advanced Async Clipboard permission:
reads, custom-format writes and writes without a gesture. Native secure-context,
focus and Permissions Policy checks remain in force. Ordinary sanitized writes
with a gesture and legacy synchronous clipboard operations retain their existing
browser policy. In the existing runtime, a sanitized-write permission query can
report Granted while the user's global Deny still blocks the actual request;
extension clipboard rules do not change that distinction. Clipboard `get` composes the rule with the same current user
permission used for admission. Source and focused unit verification pass;
Windows managed-MV3 browser acceptance passes in regular and explicitly permitted
private sessions. The 106-call page/worker matrix covers both aliases and calling
styles, including 24 private-access rejections before permission, regular-rule
restart persistence, user Deny, private rule lifetime and revocation. Native
evidence includes 54 permission queries and 11 requests with rejecting-only
clipboard items, without supplying a usable payload or reading the system
clipboard. Actual data transfer, broad iframe parity, MV2 and other platforms
remain unverified. See the implementation record for the accepted evidence.

Rules validate Chrome content-setting patterns, including schemes, subdomains,
and explicit ports. The most recently installed eligible extension takes
precedence; within an extension, primary host specificity precedes scheme and
port specificity, followed by the secondary pattern. `location` uses the
requesting document and its top frame; other implemented categories ignore the
secondary URL. File rules require the extension's current file-access grant.
Summer's website permission boundary remains HTTP(S)-only; file and FTP
permission queries report Block. File sound rules can affect native mute.
An unsupported or malformed request fails through the normal Promise rejection
or `runtime.lastError` callback contract.

Regular rules persist separately for each installation. Only active managed
extensions with the current `contentSettings` permission affect decisions.
Disabling or revoking a permission suppresses their rules; uninstall removes
them. `clear` removes only the caller's category and requested scope.
`incognito_session_only` rules require explicit private access and an active
private lifetime, override regular inheritance, stay in memory and disappear
when the last private owner closes. Native permission decisions recheck policy
and the live requesting document after asynchronous consent.
Inherited regular permission Allow becomes Ask in private sessions under
Summer's conservative consent policy; explicit private rules can grant access.

`cookies`, `javascript`, `images`, `popups`, `autoVerify`, and
`automaticDownloads` reject `get`, `set`, and `clear` explicitly. They require
capabilities or complete gesture/lifecycle semantics not established by the
existing runtime. The earlier profile-global scalar placeholders are ignored
because they contain no extension owner, patterns or scope.
Deprecated `fullscreen`/`mouselock` return their constant `allow` setting;
`plugins`/`unsandboxedPlugins` return `block`; their validated mutations are
no-ops and do not weaken browser protections. `getResourceIdentifiers` resolves
`undefined`, matching the current Chrome API; no `onChange` event is exposed.

This is a documented Summer-side subset, with no Electron fork or engine
changes. See the [Chrome contentSettings reference](https://developer.chrome.com/docs/extensions/reference/api/contentSettings).

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
`bookmarks.getRecent(numberOfItems)` accepts one required positive signed-32-bit
integer and returns URL bookmarks only, newest insertion first, with real
hierarchy parent/index fields. It uses the same authenticated, `bookmarks`
permission-checked page/worker broker and Promise/callback transport. Chromium's
schema and implementation require a count of at least one and exclude folders
(`bookmarks.json`, `bookmarks_api.cc:395-419`, and `bookmark_utils.cc:134-150`).
Summer now records an optional browser-owned `dateAdded` on genuinely new durable
URL-bookmark insertions, including imports; updates, moves and portable sync edits
preserve it. Input cannot replace that creation time. Existing records are not
backfilled: unknown or invalid dates remain the existing zero sentinel, ordered
after known dates, with equal dates following hierarchy traversal order. Creation
times are validated as non-negative safe integers. The additive desktop field
does not change portable import/sync schemas or infer historical creation dates.
Real Windows page/worker acceptance passes for recency, folder exclusion,
parent/index metadata, argument/callback errors, permission denial, mutation
preservation and restart persistence. Evidence is
`.tmp/bookmarks-browsing-data-live-evidence-parent-20261027.json`.
History operations use Summer's on-device visit store
populated from normal tab navigations. Download operations start real Summer
downloads, expose Chrome-shaped in-memory records, and route
pause/resume/cancel/icon/file operations to active Electron download items
where available. Debugger sessions attach only to regular-session HTTP(S) Summer
tabs and exact regular-session `about:blank` tabs by `tabId`. Private targets
are excluded from enumeration, attachment, commands, and event delivery even
when the extension has incognito access. Session and tab identity are rechecked
after asynchronous reservations and command completion. Other browser-owned
pages stay excluded. Sessions
are owned by the exact extension that attached them, route Chrome's
allowed CDP domains and child-session identifiers, and deliver bounded
`onEvent` and `onDetach` signals to pages or wakeable workers. Tab closure,
navigation to a different browser-owned URL, and extension unload clean up ownership.
Summer yields an internal focus-only or idle hover/wheel debugger session before
an extension attaches and retries widget focus preservation after that extension
detaches. A pressed OOPIF gesture keeps its debugger session until the matching
input-sequence cleanup so an extension cannot interrupt an active drag.
Proxy, privacy, content-settings, and font-settings calls use browser-owned
profile state. Content-setting objects expose Chrome's `get`, `set`, and
`clear` contract without inventing `onChange` events that Chrome does not
provide. Idle queries use Electron's OS idle state rather than returning a
constant result.

Enforced desktop privacy controls currently cover native WebRTC IP handling,
translation offers, browser-provided remote search enrichment, password-saving
offers, and website address/card autofill. Password/autofill settings cancel
pending requests when disabled and recheck before guarded vault commit or final
disclosure. They cannot enable private website saving/autofill or bypass normal
HTTPS, foreground, explicit-choice and card identity checks. Remaining privacy
preferences and font-rendering mutations still need enforcement; see the
[member inventory and verification record](chrome-extension-namespace-implementation.md).

The cookie, referrer, Do Not Track and hyperlink-auditing successors use Chromium's original extension
preference authority in prepared native source. Their Main adapters preserve
permission, private-consent and installation lifetimes without a JavaScript value
store. The referrer consumer covers native request metadata, page preferences
and worker preference watchers. DNT uses actual native request headers and the
window Navigator preference; auditing uses Blink's anchor-ping gate. The shared
adapter synchronizes whichever of these fixed native owners is available before
starting accepted extension workers. These additions remain uncompiled and have no
matching browser acceptance; an unavailable native owner rejects calls. Their
registry entries describe pending native integration, not verified support.

Some desktop APIs are intentionally first-pass compatibility surfaces rather
than full Chrome UI parity. Desktop `commands` accepts Chrome's string or platform-keyed manifest
shortcuts, applies platform modifier rules, and dispatches non-reserved keyboard
combinations to `onCommand`; conflicts resolve deterministically by extension ID,
while active Summer shortcuts, AltGr combinations, and `_execute_action` remain
browser-owned. `contextMenus` stores menu definitions, builds validated
`parentId` trees as native expanding submenus, substitutes `%s` selection text,
and appends only entries belonging to extensions currently loaded in the
regular tab's session. Orphaned, cyclic, and empty filtered branches fail
closed instead of being flattened into unrelated commands; private tabs never
receive the regular profile's persisted menu definitions. Clicks emit events
back to extension pages and service workers. A click on a
persistent item wakes its unloaded MV3 worker, waits for `onClicked` to
subscribe, and then delivers the event once. Summer waits for Electron's
`menu-will-close` signal and dispatches the click on a later main-loop turn so
extension delivery cannot re-enter the native menu callback. `sidePanel.open()` hosts the
declared extension page in a sandboxed native side panel that reserves browser
layout space; one panel is active at a time, and `setPanelBehavior()` can route
toolbar action clicks to it. The browser-owned `ExtensionSidePanel.vue` shell
provides a fixed header with a close control and an inline-edge resize separator.
With `openPanelOnActionClick`, the toolbar badge toggles a matching visible panel
through its close animation rather than destroying and recreating it.
Programmatic `sidePanel.open()` preserves a matching extension page in the same
window and tab scope, including its runtime connections and current pending load.
Window-wide panels remain open when an unrelated tab closes; tab-specific panels
close with their owning tab.
Web links opened by a visible side panel, including its embedded web frames, are
recreated as ordinary HTTP(S) Summer tabs in that panel's owning window. Native
popup windows remain denied; background-tab requests keep their disposition.
A dedicated sandboxed, isolated preload retargets trusted ordinary root HTTP(S)
anchor clicks to native new-tab activation. Page preventDefault still cancels the
action. Modifier/middle clicks, downloads, named targets and internal links are
unchanged. The preload has no IPC or page-world API. A MessageChannel task restores
its temporary target/rel attributes after default activation; later page-owned
attribute writes, including writes of the same value, are preserved.
Unknown external root navigation, including form POST and script-driven location
changes, remains blocked rather than being replayed as a GET. The panel document and its conversation stay retained.
Embedded-frame navigation within the provider still stays inside that frame.
Credential-bearing URLs, non-web schemes, form-POST popup requests, and requests
from hidden, closing, crashed or detached panels are rejected. New tabs retain
the normal browser navigation and HTTP-consent checks; no panel privilege or
window-opener relationship is transferred to the destination.
Remote panel child frames run the browsing session's isolated preloads, matching
ordinary tabs. The host keeps `nodeIntegration: false`, `sandbox: true`,
`contextIsolation: true`, and `webSecurity: true`; enabling subframe preloads does
not expose page-world Node or raw IPC APIs. Manifest content-script host grants
and the browser-owned preload API authorization remain separate checks.
The tab-only passkey permission wrapper and unclaimed file-drop interceptor do
not install beneath a top-level packaged extension origin: their main handlers
cannot authorize those documents. Chromium's native credential/drop behavior
and site upload handlers remain in control there. Ordinary tab guards and all
main-process authorization checks retain their existing behavior.
Tab-specific panels retain their native document separately from the window's
default panel. Switching tabs hides the departing panel and restores the returning
tab's previously opened panel without reloading its page or runtime context.
Temporary tab hiding emits neither `sidePanel.onClosed` nor another `onOpened`
when that same opened document returns. Explicit replacement and actual closure
still emit lifecycle events. This follows the [Chrome side-panel lifecycle](https://developer.chrome.com/docs/extensions/reference/api/sidePanel).

Only one panel is visible per window, while other opened tab scopes remain cached.
A default panel can be retained behind a contextual panel and is shown only where
the activated tab permits it. Closing a visible contextual panel also clears the
window's dormant default fallback, so it cannot reappear after an explicit close;
other tabs' independently opened panels remain intact. Disabled or removed panels
are disposed rather than resurrected by a later tab activation.

Tab-specific `setOptions` declarations are process-local and replace the default
declaration for that tab; they do not inherit its document path or enabled state.
Old persisted numeric tab overrides are ignored. Tab removal, extension unload,
and cross-window tab detachment clear the matching runtime overrides. A moved tab
disposes its old window-bound sidebar document and requires an explicit open in
the destination window. Window teardown and extension unload dispose all matching
retained documents. Changing options never creates an unopened panel.

### Side-panel conversation reuse

Summer's generic AI actions and the existing official ChatGPT and Claude badges
restore the current tab's own conversation beside its webpage. Generic actions
use the saved preferred extension; a named badge keeps its own extension identity.
Reuse stays within one regular browser window, prefers the current tab's scoped
panel, and never borrows another tab's scoped conversation. An extension-defined
window default is a fallback only when both the default and current tab's effective
declaration enable the same resource. If no reusable conversation remains, normal
configured-panel or extension-action opening applies. Repeated generic Open AI
focuses the panel; a named badge toggles its matching visible conversation through
the existing close animation. No additional AI toolbar button is added.

The browser restores each tab's native panel record without reassigning its tab,
window, extension runtime context, or panel declaration. A tab without its own
reusable panel follows the extension's configured-panel or ordinary action flow;
the browser does not copy another tab's declaration. Independent conversations
remain cached when their tabs are inactive, and temporary hiding or restoration
emits no synthetic extension close/open events. Closing tab A disposes A's scoped
panel while tab B's own document, draft, and runtime connections remain alive.

The shell close control closes the actual displayed source conversation and
removes every tab selection referring to it. Those selections suppress older
fallback panels, preventing an explicitly closed conversation from reappearing
on tab return. Native scoped/default close behavior continues to clear its
default fallback. Removing a tab disposes its scoped conversation and selections;
unloading the extension or tearing down the owning window disposes all its panels. Reuse
never crosses windows or private-session boundaries. Ordinary programmatic
`sidePanel.open()` retains its existing scope and open behavior.

Opening expands from the docked edge over 180 ms; reduced-motion preferences
disable the transition. The close button and extension-requested closes (such
as the toolbar badge) share a 180 ms collapse before closing the native document;
reduced motion closes immediately. Extension-requested animation completion is
acknowledged by the owning browser window with a per-request token; stale
acknowledgements cannot close replacement panels, and an unresponsive renderer
falls back to closing after 400 ms. Unload and window teardown remain immediate. Native content
bounds follow the animated layout width in both directions.
Drag the separator or focus it and use Left/Right (Shift for larger steps),
Home/End for width limits, or double-click to reset. Width is remembered within
the window across panel closures, capped at 600 pixels, and constrained to leave
webpage space on narrow windows. The native extension view occupies only the
content rectangle below the header and outside the resize gutter; renderer
stacking cannot cover a native child view. Bounds still pass the existing
owner-window validation in the main process. While the navigator is visible
(including its exit animation), native side-panel content stays below the
browser overlay. This only changes view ordering: the extension document stays
attached with its session and size, and no extension open/close events fire.
The renderer paints only the panel header, border and resize gutter; its native
content rectangle remains transparent. Otherwise raising browser chrome for a
new address-bar query would blank the entire panel, including areas outside the
navigator. Dismissing the navigator restores the same native panel above browser
chrome with its draft and input intact.
Wallpaper and widget visibility do not lower the panel. Native popup controls
retain their upper layer. Focused coverage lives in
`tests/extensionSidePanelUi.test.ts` and `tests/extensionSidePanels.test.ts`.
`playwright/ai-panel-tab-ownership.spec.ts` exercises the official AI badge route
with a synthetic same-origin extension: independent drafts, switching both ways,
closing inactive tab A while B stays focused, and B's live connection afterward.
This does not authenticate a real provider account.
`playwright/extension-side-panel-tab-return.spec.ts` verifies retained native
documents, drafts, runtime identities, native content transparency while the
navigator is raised, input after dismissal, and explicit close behavior with a synthetic
extension in its own disposable browser profile.
`playwright/extension-panel-storage-parity.spec.ts` verifies child preload
availability, sandbox/Node/IPC isolation, and granted versus ungranted host
storage/cookie behavior. Its ChatGPT-origin child uses intercepted static HTML
and an unknown authorization nonce; it does not establish live ChatGPT loading.

`getPanelBehavior()` reads that extension's saved
click behavior, defaulting to false, through the same authenticated page and
worker broker. It returns a fresh object and does not open a panel. Windows MV3
page/worker acceptance verifies process-restart persistence, another extension's
independent settings, held/current API references after extension installation,
argument and permission refusals, and Promise/callback error shape. Side-panel
callbacks now report errors in the public page runtime; worker setters resolve
undefined and error callbacks receive no arguments. Private execution remains
unavailable. Evidence is `.tmp/side-panel-behavior-live-evidence-parent-20261027.json`;
this does not establish complete side-panel namespace or cross-platform parity.
`sidePanel.getLayout()` reports the physical right edge. Lifecycle deliveries
capture the creating installation and cannot transfer a retained document's
historical events to a replacement installation. The retained document and
visibility behavior above supersedes earlier single-panel acceptance records.

`omnibox` recognizes a manifest
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
Deferred dynamic content-script and user-script phases retain the exact frame,
URL and document generation observed when scheduled. Tab destruction, detached
frames and replacement navigation (including same-URL reloads) retire pending
work. Closure while discovery, approval or world preparation is pending cannot
move that work to a replacement document; unexpected phase failures are reported
without logging script contents or page data.
`identity.getRedirectURL(path?)` returns a string synchronously in extension
pages and workers, following the [Chrome identity contract](https://developer.chrome.com/docs/extensions/reference/api/identity#method-getRedirectURL).
Its fallback concatenates a bounded path after the native extension's fixed
`https://<id>.chromiumapp.org` host; it performs no IPC, authentication or network
request. Existing callable native bindings are preserved. Other identity methods
retain the authenticated broker and its permission checks.
Generic `identity.launchWebAuthFlow()` runs in a fresh in-memory, sandboxed
session and resolves only an exact extension redirect origin; Google-account
token APIs remain unavailable. Native messaging resolves a
platform-installed Chromium-family host manifest, verifies its exact extension
allowlist and executable path, and uses bounded Chrome framing. On Windows,
a small two-line `.bat` or `.cmd` wrapper that starts an absolute executable
with one absolute script and forwards the browser arguments is resolved to a
direct launch; arbitrary batch commands are rejected. One-shot calls
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
atomic same-window and cross-window movement between owned normal windows of the
same privacy/session class. Batch transfers stage native view ownership with
rollback before publishing tab order, ownership and group identity together;
attachment events observe the committed group, and an emptied source closes later.
Chrome-style pinning is main-owned, lasts for the native tab lifetime, and keeps
pinned tabs before ordinary tabs through creation, dragging and transfers. Pinning
ungroups a tab; grouping unpins it. Its visible keyboard-accessible unpin control
is separate from Summer's persistent Keep behavior. Automatic discard eligibility
is not implemented; enabling `autoDiscardable` rejects rather than reporting a
successful state change.

Tab `width` and `height` describe the exact owned content view, including a
legitimate zero-sized view, rather than the decorated OS window. Native popups
use their own content area. Missing, stale, foreign-window or mismatched private-partition
geometry returns zero; it never borrows another tab's size. Window API dimensions
continue to describe the outer window. Positive container bounds alone do not
prove page rendering or screenshot readiness.

On the declared Electron 44.1 runtime, inactive browser tab views stay attached
to their exact owning window with native visibility disabled. A new unparented
view receives the exact owning chrome's measured, size-matched positive viewport
bounds before its first hide when that layout is cached; missing-layout readiness
is a separate acceptance case. Creation does not
select or focus them; selecting a tab hides the previous page before revealing
the selected page through the existing stacking/fullscreen path. Switching uses
constant native view operations. Transfers hide pages before native ownership
changes, and rollback restores every native parent, order and bounds before
restoring visibility. Native BrowserWindow popups retain their own lifecycle.
This uses existing window/session checks and changes no renderer IPC authority.
Native visibility and screenshot readiness are distinct from the document's
Page Visibility API: Summer does not spoof document visibility or claim Chrome
background throttling parity. Capture in a wholly hidden OS window remains a
separate unverified/failing diagnostic case.

The native screenshot contract in `extension-created-tab-viewport.spec.ts` uses
the exclusive test project and requires the existing Windows visible-window
opt-in. Run its two cases with process-level `SUMMER_E2E_WINDOW_MODE=background`
and `SUMMER_E2E_ALLOW_VISIBLE_WINDOWS=1` for non-activating native windows.
The default hidden gate skips these pixel assertions; positive geometry and
document visibility do not establish a native screenshot surface.

Document-targeted `tabs.sendMessage` preserves exact-document delivery by mapping
Summer UUIDs to native Chromium UUIDs and revalidating the current frame, document,
tab/session and extension authority before delivery. Native read-only identity
resolution is preferred; where unavailable, a fixed isolated scripting identity
probe additionally requires the extension's current scripting permission. Missing
identity authority fails closed. No document ID is dropped for a frame-wide or
MV2 broadcast fallback.
Regular extension pages and service workers with `tts` permission share one
browser-owned speech queue. Native local voices and accepted extension engines
are available through callback and Promise queries. `ttsEngine` requires its
required manifest permission; manifest `tts_engine.voices` and dynamic
`updateVoices` registration are installation-owned. Eligible engines register
both speak and stop listeners. Utterance callbacks target one current document
or worker generation and are revoked on completion, cancellation, permission
loss, navigation, worker replacement or unload. Dynamic voices survive disable
and restart; uninstall removes them. Speech text and callback capabilities are
never persisted. Native Windows fallback uses local System.Speech when Electron
has no local Web Speech voices; other platforms use the isolated native Web
Speech host and remain unverified here. Native prosody depends on the OS voice;
full SSML interpretation and OS voice-catalog change observation remain gaps.

General settings contains a speech-language manager for engines implementing
the stable install, uninstall and status events. Only Summer browser chrome can
issue those language requests. The UI displays actual engine status updates;
successful request delivery does not imply installation succeeded. Speech and
language management are unavailable in private windows. ChromeOS-only audio
stream synthesis remains excluded. See the namespace implementation record for
quotas and current verification evidence.

Regular-profile extensions with the effective `printerProvider` permission can
register printer discovery, CDD capabilities and print-job callbacks. Summer's
browser Print command and unmodified Ctrl+P (Cmd+P on macOS) open a browser-owned
destination/settings dialog when accepted providers are present. User-configured
shortcuts retain priority. Private windows and profiles without providers retain
the system dialog; the dialog also offers an explicit system-print action.
Website `window.print()` still uses the system print path.

Selecting a destination does not share document bytes. Explicit Print generates
the current tab's PDF using its selected media/orientation, then delivers a real
recipient-realm Blob to the selected provider. PDF is preferred when advertised;
PWG-only printers receive genuinely rasterized PWG data, never renamed PDF.
The rasterizer uses bundled PDF.js in an isolated, sandboxed, no-preload,
network-denied renderer. Its ephemeral session accepts only fixed bundled
assets; page pixels are streamed into bounded PWG row compression. sRGB8/sGray8,
advertised DPI and duplex sheet-back transforms are honored without silently
reducing the requested resolution. Over-budget jobs fail explicitly.

CDD/CJT support covers media size, orientation, color, DPI, copies, duplex,
collation and reverse-order settings. Hardware settings stay in the ticket;
unsupported rendering fields, including page ranges and vendor ticket items,
are rejected. Documents are limited to 64 MiB, raster pages to 40 million pixels
and 200 pages, and raster DPI to 72–600. Discovery, capability and job callbacks
have 10-second, 15-second and 120-second limits. Native PDF generation has a
60-second limit; retries cannot queue more PDF work while a native render remains
unsettled. Printer selections belong to the actual browser document;
callbacks and sequential chunk tokens belong to one native provider document
or worker generation. Navigation, active-tab changes, close, permission loss,
provider replacement and shutdown revoke related capabilities and release tasks.
Printer lists, titles, capabilities and documents are not persisted. “Sent to
printer” means the provider returned `OK`, without claiming physical output.
The ChromeOS USB identification event remains deferred. Windows native page and
worker delivery, actual PDF/PWG jobs, provider-error retry, reload cancellation
and stale-callback isolation passed managed Summer acceptance. Independent
native raster decoding also verified pixels, duplex transforms and cleanup.
Other platforms, native MV2 and long idle suspension remain unverified; see the
namespace implementation document for the exact acceptance record.

Runtime compatibility initialization does not rewrite approved extension
snapshots. Its cost is bounded to the registered frame, service-worker preload,
or already-reviewed Manifest V2 bootstrap and the APIs requested by that
extension context.

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
| Worker cookies, bookmarks, history, downloads, alarms, notifications, windows, settings, and DNR | Calls and events use the authenticated worker relay; cookie access also requires a matching HTTP(S) host permission. Summer keeps its twelve DNR broker methods on the per-context native cache when its shape permits, retaining one browser-owned matcher across binding refresh. | First-class Summer broker. | Give pages and workers the same supported semantics through one authenticated, permission-checked relay, including worker wake-up, lifetime, cancellation, and cleanup. | `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| Runtime and external messaging without a complete native pair | Same-extension one-shot and port messaging use the authenticated browser-owned router, with bounded payloads, reply lifetimes, worker wake-up, disconnect, and extension cleanup. Extension-to-extension calls follow Chrome's manifest default: every extension may connect when `externally_connectable` is omitted, while a present key restricts callers to `ids`; one-shot messages dispatch through `onMessageExternal` and ports through `onConnectExternal`. Regular top-level HTTP(S) pages receive only a narrow `runtime.sendMessage()` method, and the browser process accepts it only when the target's validated `externally_connectable.matches` covers the authenticated current page URL. | First-class Summer broker; page-owned values select a target but never grant authority, ordinary website child frames and private-profile pages cannot use the website bridge; a direct HTTPS child of its live, visible owning extension panel may message that extension without a sender tab, and no page-owned transport carries browser API authority. | Retain browser-owned sender authentication, bounded replies and worker wake-up while expanding external message surfaces only when their manifest declaration and profile boundary can be enforced exactly. | `electron/preload/externalExtensionMessaging.ts`, `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| Optional permissions | Required permissions and hosts are active immediately; `optional_permissions` and `optional_host_permissions` remain inactive until `permissions.request()` receives explicit native user consent. Exact and wildcard-covered narrower host grants persist per extension, affect browser-process API and host authorization, can be revoked through `permissions.remove()`, emit wakeable `onAdded` / `onRemoved` events, and are erased when the extension is removed. | First-class browser-owned consent boundary; undeclared requests, required-permission removal, malformed input, and denied prompts fail without changing grants. | Keep grants constrained to the currently declared optional set, serialize concurrent mutations, and require browser-owned consent before widening extension authority. | `electron/main/extensions/extensionChromeApis.ts`, `electron/preload/extensionApiShim.ts` |
| Alarms | Bounded alarms persist in browser-owned storage, are restored only for active managed extensions that retain the effective permission, fire overdue one-shots after restart, and advance repeating schedules before dispatch. | First-class persistent scheduler with per-extension quotas and lifecycle reconciliation. | Retain durable mutation-before-event ordering, worker wake-up, and shutdown-safe timer cleanup. | `electron/main/extensions/extensionAlarms.ts` |
| Tabs, frame navigation, and general read-only services | `tabs.move` transfers individual tabs within or between regular Summer windows of the same privacy class, emits `onDetached` / `onAttached`, and reconciles both tab strips; per-origin zoom methods/events, OOPIF-safe frame-tree webNavigation, MHTML page capture, non-private recently closed tabs, top sites, and bounded CPU/memory/storage information use browser-owned state. | First-class where Summer owns an exact source; unsupported navigation target events remain explicit gaps. | Add a surface only when its identity, ownership, and lifecycle can be represented without inventing Chrome state. | `electron/main/front/tabs.ts`, `electron/main/extensions/extensionBrowserHost.ts`, `electron/main/extensions/extensionWebNavigation.ts`, `electron/main/extensions/extensionPageCapture.ts`, `electron/main/extensions/extensionSessions.ts`, `electron/main/extensions/extensionSystemInformation.ts` |
| Capture, tab groups, and TTS | Desktop/tab capture uses a browser-owned bounded picker and Electron media IDs; pending/active/stopped/error tab-capture state is authenticated and observable through `getCapturedTabs()` and `onStatusChanged`; tab groups own metadata, membership and atomic transfers between same-session normal windows; extension-page TTS uses Web Speech. | Bounded compatibility surfaces; captured-tab `fullscreen` remains false and worker TTS has no speech engine. | Add only state Summer can observe exactly, and keep raw Electron capture authority out of extension contexts. | `electron/main/extensions/extensionCapture.ts`, `electron/main/extensions/extensionTabGroups.ts`, `electron/preload/extensionApiShim.ts`, `electron/preload/extensionTts.ts` |
| Native messaging | Chrome, Chromium, Edge, and Brave native-host manifests and allowlists are validated before one-shot or persistent framed processes start; ownership, process count, writes, replies, and teardown are bounded. | First-class broker for platform-installed compatible hosts; host installation/discovery UI is outside Summer. | Retain exact executable and origin validation and never expose child processes or filesystem paths to extensions. | `electron/main/extensions/extensionNativeMessaging.ts` |
| `storage.session` | Summer preserves Electron's complete native MV3 area, including default content-script denial, trusted-page grant/revocation, shared values, and scoped change events. A complete native area must expose every promised method plus `onChanged`; otherwise trusted pages and workers receive Summer's authenticated in-memory fallback while content-script access fails closed. | First-class native Chromium binding on the pinned runtime, with a bounded Summer fallback for trusted extension contexts. | Keep Chromium as the authoritative isolated-world owner while it passes the completeness and E2E contract; never replace it with a DOM relay or manifest-widening bootstrap. | `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionSessionStorage.ts`, `playwright/extensions-manager.spec.ts`, `docs/chrome-extension-content-script-session-storage.md` |
| Worker `storage.sync` | The browser-owned provider enforces Chrome-shaped quotas and scoped events; paired devices can opt in to Mesh transport. | Mesh opt-in on both hosts, active signed extensions and foreground execution are required. | Verify installed-extension workflows separately from protocol and broker tests; see [Mesh storage](extension-mesh-storage-sync.md). | `electron/main/extensionStorageSyncHost.ts` |
| Page/worker `storage.managed` | Summer's bounded read-only OS-admin JSON provider supplies `get`, `getKeys`, zero-valued `getBytesInUse` and managed change subscriptions. Byte queries validate Chrome's selectors; `set`, `remove` and `clear` reject with the read-only-store error. | Windows regular MV3 pages/workers pass the original 272-call baseline and a 528-call follow-up, including current/held methods, real second-extension binding refresh, selector errors, permission denials and manager lifecycle. Empty policy, controlled defaults and no false change events are verified; schema-filtered populated policy, positive change delivery, content scripts, private execution, MV2 and other platforms remain unverified. | Keep native storage-container ownership stable, preserve other areas and authorize reads/events again before releasing data. Native provider/schema integration and `managed.setAccessLevel` remain unfinished. | `electron/preload/extensionApiShim.ts`, `electron/main/extensions/extensionManagedStorage.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| Context menus without an authenticated relay | Eligible extension pages and workers use the authenticated context-menu host with persistent worker wake-up and extension cleanup; unsupported contexts keep the namespace absent. | First-class Summer broker where an authenticated relay exists, otherwise fail closed. | Route every eligible page and worker through the authenticated context-menu host with worker wake-up and cleanup; otherwise keep the namespace absent. | `electron/preload/extensionApiShim.ts` |
| `idle` | Queries use the OS idle state, while one shared browser-owned monitor emits threshold-aware `onStateChanged` transitions and cleans up with the extension lifecycle. | First-class cross-platform OS-backed query and event producer. | Add a cross-platform OS-backed transition producer for `onStateChanged`, with one shared poll/subscription source and extension lifecycle cleanup. | `electron/main/extensions/extensionChromeApis.ts` |
| Browsing-data removal | Cookies, Summer history, and Summer download records honor their supported scopes; cache rejects unsupported scoping. Password and autofill-profile removal is all-time and profile-wide, preflights every requested category, names the requesting extension and categories in a localized native warning, rejects cancellation, verifies password removal, and invalidates open Settings state. Payment cards are never deleted. | Honest bounded subset with preflight validation and explicit user confirmation for sensitive profile-wide deletion; no false success for cancellation or data Summer cannot prove it erased. | Implement each category only with the exact scope Summer owns; retain confirmation for password/profile deletion, keep payment cards out of `formData`, and reject unsupported or scoped requests before any deletion. | `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionBrowsingData.ts`, `electron/main/credentialStore.ts`, `electron/main/autofill.ts` |
| Blocking `webRequest` | Event-specific filters and extra-info options are validated in the browser process; protected request and response headers require `extraHeaders`, blocking results are authenticated to their exact extension context, terminal DNR actions run before `webRequest`, DNR header changes retain priority, cross-extension conflicts use stable extension-ID ordering, and `handlerBehaviorChanged()` has Chrome's 20-call-per-10-minute quota. | First-class bounded broker; unsupported schemes or fields reject, responses fail open after 500 ms, and protected headers that were withheld from a listener survive full-list replacements. | Add resource types, schemes, and security information only when Electron can identify and expose them without widening the listener's authority; retain authenticated responses, deterministic conflicts, and bounded timeouts. | `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionDeclarativeNetRequest.ts` |
| DNR rules, match feedback, and action options | One bounded browser-owned decision combines separately compiled static and mutable matchers; active/standby worker generations keep the previous rules serving until a replacement is complete. Deprecated initiator aliases and top-frame domains use authenticated request context, session rules honor Summer tab filters, request-method exclusions work across scopes, validated direct, extension-path, regex, and full URL-transform redirects are supported, and extension-path redirects use Chromium's runtime `dynamicId` when the manifest resource requires `use_dynamic_url`. Response-header include/exclude conditions can make late block or redirect decisions, request and response header actions use per-extension priority with stable extension-ID ordering, before-request decisions select one candidate per extension and resolve block before redirect or upgrade before allow, and feedback returns a five-minute match history under Chrome's 20-call-per-10-minute quota. Matched header rules are included, action-count options reject missing tabs or disabled badge updates atomically, allow rules do not inflate the count, positive signed-32-bit rule priorities are preserved, and Chrome-shaped static-ruleset, mutable-rule, unsafe-rule, and regex-count limits are enforced atomically under Summer's stricter 150,000-static-rule cap. Extension-controlled patterns use the shared linear-time `re2js` engine with ASCII validation, capture-aware substitution, and a Chrome-shaped 2 KiB compiled-program cap. Chrome's `webbundle` and `webtransport` values are retained, while Electron-unobservable requests restricted to either type remain unmatched. Main-frame `allowAllRequests` state follows the Summer tab across Electron's navigation-time frame replacement; sub-frame state remains scoped to its authenticated frame hierarchy. Both are cleared on navigation, tab removal, or extension changes. | First-class bounded broker; unsupported, unsafe, or incorrectly scoped rules reject instead of being widened. | Retain Chrome-valid resource types without claiming classifications Electron cannot observe; keep RE2 syntax and compiled-program compatibility covered when either Chrome or `re2js` changes. | `shared/chromeDeclarativeNetRequest.ts`, `electron/main/extensions/extensionDeclarativeNetRequest.ts` |
| `identity` account/OAuth methods | Generic web-auth flows use a fresh nonpersistent session with downloads, permissions, popups, webviews, Node, and DevTools denied; only the authenticated extension's exact `chromiumapp.org` redirect resolves. Redirect URL calculation and empty local profile info remain available. Google-account token calls still reject. | First-class isolated generic OAuth flow; browser-account token APIs require an account provider Summer does not have. | Add token APIs only with a user-consented browser account provider, scoped token storage, revocation, and clear account UI. | `electron/main/extensions/extensionIdentity.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| `offscreen` | Desktop: Chromium/Electron owns MV3 create/close/has, document isolation and lifetime; Summer no longer provides a hidden-document host. This unlisted Electron API is provisional, not a full-support guarantee. Mobile V56: required-permission MV3 callers get Promise-only create/close/has for exact `WORKERS`, one PAGE document per extension/eight globally, a six-member provisional runtime plus non-enumerable `lastError`, verified package modules/Workers, and process-only teardown under strict CSP/navigation/network bounds. Other mobile reasons and suspended/killed-app execution are absent. | Desktop native provisional API; mobile bounded compatibility host. | Keep the desktop native lifecycle contract passing on Electron upgrades; reason-specific audio timeout, crash recovery and all-platform parity are not established by the DOM-parser test. Expand mobile reasons only with reason-specific sandbox and lifetime enforcement. | `playwright/extension-offscreen.spec.ts`, `shared/chromeExtensionCompatibility.ts`, `packages/summer-android/src/extensions/mobileExtensionPackage.ts`, `packages/summer-android/android/app/src/main/java/com/hiketech/summerbrowser/SummerTabsPlugin.java`, `packages/summer-android/ios/App/App/SummerTabsPlugin.swift` |
| `userScripts` | Explicitly user-approved registrations execute in isolated worlds on reviewed hosts. A frame-preload handshake gates every injection, installs each configured CSP before execution, and exposes only permission-bounded runtime one-shot/port messaging when `messaging` is enabled. | First-class bounded isolated-world bridge; every call is re-authorized against the live frame, current host grant, matching registration, extension state, and user approval. | Retain fail-closed authorization, bounded messages/ports, and configuration-before-injection ordering across every injection phase. | `electron/main/extensions/extensionUserScripts.ts`, `electron/preload/extensionUserScriptWorld.ts`, `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionManager.ts` |
| Desktop `commands`, `omnibox`, and `sidePanel` | On desktop, string or platform-keyed shortcuts are normalized with Chrome's modifier rules, active Summer accelerators and AltGr combinations remain unavailable, conflicts resolve by stable extension ID and appear unassigned through `getAll()`, omnibox deletion/disposition events are complete, and default or tab-specific native panel state is wired into Summer UI; tab panels hide and restore with tab activation. | First-class Summer integrations with documented bounds rather than Chrome UI emulation. | Add a user-facing browser-owned shortcut remapping surface before claiming Chrome's shortcut-management UI; retain deterministic conflicts, native panel state, and consistent keyboard and accessibility behavior. | `electron/main/extensions/extensionChromeApis.ts`, `electron/main/extensions/extensionSidePanels.ts`, `ui/store/suggestions.ts` |
| Native messaging | `sendNativeMessage()` and `connectNative()` discover platform-installed Chrome, Chromium, Edge, and Brave manifests, require the exact extension origin in `allowed_origins`, canonicalize the executable, and launch without a shell. Linux discovery also checks those browsers' Flatpak manifest directories. A bounded Windows `.bat`/`.cmd` wrapper that only invokes an absolute executable and script with `%*` is resolved to a direct launch; other batch files fail closed. One-shot calls use bounded framing and a 30-second deadline; persistent ports authenticate ownership, enforce global/per-extension process and pending-write limits, deliver bounded framed messages, and terminate on disconnect, extension unload, or shutdown. A worker-owned native port holds an authenticated native worker task until it closes. Host disconnect delivery hands that task off through an exact-owner acknowledgement after synchronous listeners run; closing ports remain quota-counted until acknowledgement or a five-second cleanup deadline. Local disconnect, worker stop, unload, shutdown, and failed discovery release the task exactly once; pending discovery keeps its separate admission reservation until it settles. | First-class bounded transport with strict host ownership, framing, backpressure, and lifecycle cleanup; native-host installation and discovery UI remain platform-owned. | Retain authenticated ownership and guaranteed process termination while keeping host discovery limited to compatible Chromium-family registrations. | `electron/main/extensions/extensionNativeMessaging.ts`, `electron/main/extensions/extensionChromeApis.ts` |
| File-scheme access | Disabled by default. For an extension whose reviewed host declarations include file URLs, the manager can grant or revoke access; Summer reloads the extension with the matching Electron `allowFileAccess` setting and the authenticated status method reports the active grant. | Explicit browser-owned per-extension grant tied to reviewed manifest access. | Keep file access disabled by default, require an explicit user decision, revoke stale grants when reviewed file-host access disappears, and never infer the grant from a host declaration alone. | `electron/main/extensions/extensionManager.ts`, `electron/main/extensions/extensionChromeApis.ts`, `electron/preload/extensionApiShim.ts` |
| Incognito access | Disabled per extension by default. Extensions Manager can explicitly allow or revoke access; `isAllowedIncognitoAccess()` reports that browser-owned grant. Allowed extensions are loaded separately into the disposable private session while at least one private window exists. Granting access refreshes only matching private tabs, and private toolbar actions and popups stay in that isolated session. | Bounded split runtime. Electron requires a path-backed persistent session to load an extension, so Summer uses a random temporary directory, unloads extensions, clears Chromium data, attempts whole-directory deletion after the last private window, and reclaims locked dead-process leftovers at startup. Reuse is blocked only when neither logical clearing nor physical removal succeeds. | Retain explicit per-extension consent and isolated lifecycle cleanup; adopt a true in-memory extension host if Electron provides one. | `electron/main/extensions/extensionManager.ts`, `electron/main/extensions/privateExtensionSession.ts`, `electron/main/front/privateSessionStorage.ts`, `electron/main/front/sessions.ts`, `electron/main/extensions/extensionActions.ts` |
| IDs without `crypto.randomUUID` | Browser or Node UUIDs are preferred; supported older contexts use `crypto.getRandomValues`, and identifier generation rejects when secure randomness is unavailable. | Cryptographically random but still non-authoritative identifiers. | Use browser-provided UUIDs in every supported context, or a cryptographically random browser-owned fallback; identifiers remain non-authoritative. | `electron/preload/extensionApiShim.ts` |

## AI extension compatibility verification

### Replacement listeners and native closed shadow roots

`tabs.onReplaced` supports authenticated listener registration, duplicate
suppression, membership queries and removal in extension pages and workers.
This is limited listener compatibility: Summer has no browser-owned prerender
or atomic tab-replacement producer. Navigation, reload, renderer/frame changes
and close/new-tab sequences must not manufacture `(addedTabId, removedTabId)`
events. Raw Electron event IDs are not forwarded as Summer tab IDs. Chrome's
[event contract](https://developer.chrome.com/docs/extensions/reference/api/tabs#event-onReplaced)
requires an actual replacement lifecycle.

The pinned Electron `44.5.1+wvcus` already supplies native
`chrome.dom.openOrClosedShadowRoot` in genuine extension content-script worlds.
The direct live boundary probe confirms a synchronous real closed `ShadowRoot`
and its contents through native ISOLATED execution. The method is kept native:
Summer does not synthesize an open-root-only getter, intercept `attachShadow`,
or project DOM access into ordinary website MAIN worlds. Chrome's
[DOM API contract](https://developer.chrome.com/docs/extensions/reference/api/dom)
returns the real root synchronously; an IPC/CDP snapshot cannot substitute for it.

Extensions separately granted `debugger` can also inspect a closed-root fixture
asynchronously through `DOM.getDocument({depth: -1, pierce: true})`. That path
retains exact target/extension ownership and rejects commands after detach.
It neither implements nor grants the synchronous DOM API. The dedicated
`playwright/extension-remaining-api-boundaries.spec.ts` records these boundaries.

The signed ChatGPT package is exercised through the desktop manager, real
toolbar action and side-panel host. Installation and background startup are
separate from authenticated AI browser-control acceptance. Commands, observed
versions, failures and remaining checks are recorded in
[AI extension integration tests](ai-extension-integration-tests.md).

Service-worker debugger events now use the authenticated broker. Debugger target
discovery returns actual Chromium IDs only after mapping them back to current,
regular-profile browser contents. Registered website popups keep their existing
native UI and actual window/content identities; they do not become extension
popups. Private navigation is excluded from the normal-profile navigation
monitor and event delivery. A created-navigation-target event requires an actual
native opener and a successfully registered target.

Summer's navigation document IDs and Chromium's native scripting IDs have
different formats. Document-targeted scripting therefore uses a browser-owned
snapshot of the requested live frames, a harmless native isolated-world probe to
obtain their exact native document IDs, and another authenticated generation
check. The original script is executed against native **document IDs**, never
downgraded to frame-ID targeting. Results map back to the corresponding Summer
document identities. The native probe and actual payload both retain Chromium's
permission and host checks. Navigation, missing IDs, cross-tab identities and
replacement WebContents fail closed. The extra probe is deliberate: it prevents
a document ID from being silently redirected to a newer document in the same
frame. The executable regression is
`playwright/extension-document-identity.spec.ts`.

`runtime.getContexts()` merges native background/offscreen records with live
browser-owned sidebar records, then applies the caller's bounded filter. Sidebar
identities are opaque IDs for committed same-extension documents, retained across
same-document navigation and removed on destruction or renderer loss. They do
not authorize scripting. In-place sidebar path changes navigate the existing
view without artificial close/open events; actual user closure still emits the
latest document path.

Native messaging retains a per-port correlation token before the asynchronous
connect response, so an early greeting cannot be lost or confused with another
port. Early buffering is capped at eight frames and 1 MiB per port. The token
correlates internal events; it never replaces extension/sender authorization.
Callback-scoped errors remain available during disconnect handlers and are
cleared afterward. Missing hosts use Chrome's exact error string so unchanged
provider code can select its normal fallback.

Startup and verified access-neutral update notifications now use the same
authenticated lifecycle queue as installation. A registered update listener can
defer activation until reload or profile restart. The retained candidate is
revalidated against the signed download's content address before activation;
invalid candidates are cleared without disabling the current runtime. Updates
that expand permissions still require the existing browser-owned review.

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
declared-access changes during review. Any required browser-owned Manifest V2
context-menu bootstrap is added to the staging copy before its final hash and
review; approved runtime snapshots are never rewritten during load. Extensions
are loaded with local `file://` access disabled.

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
manifest key conflicts with its signature. The same pre-review staging phase
adds the versioned Manifest V2 context-menu bootstrap when required. ZIP paths,
symbolic links, unsupported compression, invalid CRCs, excessive file counts,
and bounded package or extracted-size violations are rejected. Summer currently
accepts packages up to 512 MiB, CRX extraction up to 1 GiB, and unpacked
snapshots up to 512 MiB so large store extensions still install without
removing ZIP-bomb guardrails. Package
files are extracted into Summer's browser-owned profile storage; the original
local CRX is never executed in place.

Unpacked folders are copied into browser-owned staging, receive any required
versioned Manifest V2 bootstrap, and are hashed before the immutable snapshot's
review is confirmed. Electron loads that reviewed snapshot, not the mutable
source folder, so an installed extension keeps working if the original source
folder moves or disappears. Selecting the same source folder again creates a
new snapshot and is treated as an update; expanded declared access requires a
new permission review before the snapshot replaces the installed copy. Snapshot
replacement is atomic, and obsolete browser-owned snapshots are cleaned without
touching user files. When the source manifest has no developer-provided `key`,
Summer adds a browser-owned identity key derived from the canonical source path
to the snapshot only. This keeps the Electron extension ID, storage origin, and
grants stable across content-addressed snapshot replacements without modifying
the user's source or treating the generated key as an authorship signature.

Extension-owned pages do not inherit Summer's website permission defaults or
website preload capabilities. Summer's registered session preload exits before
installing `window.summi`, autofill probes, overlay IPC, or browser key
propagation on `chrome-extension://` pages. Extensions are never loaded into
Summer's browser-interface session.
## Browser-owned API lifecycle

Incoming native worker Ports now use the same browser-authenticated sender-tab
classification as native one-shot messages. Electron can attach synthetic tab
metadata to offscreen WebContents; Summer removes it only after the main process
confirms the exact live same-extension document is outside its actual tab and
popup registries. Real registered tabs keep their translated ID; unknown or
failed classification preserves native metadata. Native methods remain bound to
the native Port, and callbacks receive the public Port consistently. During the
bounded two-second classification, startup traffic is buffered up to 64 messages
and 1 MiB; overflow disconnects the exact Port. Native disconnect errors are
captured inside Chromium's callback and exposed as scoped `runtime.lastError`
only during the delayed disconnect callback, with previous descriptors restored.
This changes no provider package or website authority.

Desktop offscreen documents are owned by Chromium/Electron, not a Summer
hidden-window implementation. Electron treats unlisted extension APIs as
[provisional](https://www.electronjs.org/docs/latest/api/extensions/).
`npm run test:e2e:run -- playwright/extension-offscreen.spec.ts` exercises the
native DOM-parser document, worker messaging, restricted API surface,
single-document rule, close/recreate and extension-unload teardown. A separate
fixture creates an offscreen document for `USER_MEDIA` and `CLIPBOARD`, then
records nonempty `audio/webm` chunks with `getUserMedia` and `MediaRecorder`
using a disposable fake microphone and test-only automatic media permission.
This does not verify the real microphone permission flow. An
extension page with the reviewed `offscreen` permission can send a one-shot
message bearing `__ocic_offscreen: true` directly through Chromium's native
runtime channel to its offscreen document. Other extension-page messages use
Summer's broker; generic extension-page-to-offscreen messaging remains
unavailable. Keeping these routes exclusive prevents a no-response worker
listener from receiving the same message through both transports.
The preload snapshots that native delegate before augmenting Chromium's cached
runtime namespace; later namespace publication must not replace the delegate
with Summer's own wrapper. Reuse remains bound to the same extension document
and its reviewed `offscreen` permission.
Native offscreen documents use a bounded synchronous clipboard fallback when
Chromium's `document.execCommand("copy")` returns false. The fallback copies
only selected text from the top-level document of an installed MV3 extension
with effective `offscreen` and `clipboardWrite` permissions; other commands
retain their native behavior. A disposable fixture verifies its textarea copy
result and the text sent to Electron's clipboard API without changing the OS
clipboard during the test. The upstream recorder itself has not been run
end-to-end in Summer. This suite does not cover every reason, audio lifetime,
renderer crash or operating system.

Extension host-permission CORS grants apply only to authenticated privileged
extension origins, never manifest-sandboxed/opaque origins. Sandboxed pages use
the server's ordinary CORS response, even when their package has host access.
The navigation-error compatibility fallback retains actual pending main-frame
request metadata; it does not invent a request ID or assume `GET`. Unobserved,
ambiguous or expired correlations are left to native `webRequest` delivery.

Summer's browser-owned Chrome API host normalizes tabs and windows before they
cross the extension boundary. It supports tab lookup, query, creation, update,
reload, removal, and the corresponding create/update/activate/remove events.
Tab queries support URL patterns and normal/popup window filtering. Extension
pages can open their declared options page with `runtime.openOptionsPage()`.
Window reads and focus/removal events cover only windows in the current Summer
profile process. An extension may create one same-extension page in a popup
window and later focus or resize that popup. It may also resize normal windows
it created; normal windows created elsewhere may only be focused. Individual tabs may move between regular windows
of the same privacy class with Chrome-shaped detach/attach events. Whole tab
groups transfer atomically between owned normal windows of the same session,
preserving group metadata. Cross-extension popup navigation and arbitrary
browser-window mutation remain rejected explicitly.

`tabs.sendMessage()` keeps Electron's native delivery for ordinary Summer tabs
and their content scripts. Tabs returned by browser-owned `windows.create()`
popups use their actual native `WebContents.id` and an authenticated Summer
route selected from Main's registered target kind, rather than a numeric range.
Electron does not register arbitrary extension popup pages in Chromium's tab
messaging registry. That route accepts only the extension that owns the live
registered popup, targets only subscribed frames belonging to that exact popup,
and accepts a response only from a frame that received the message. Closing the
tab, window, or extension settles pending calls, and request and response
payloads retain the runtime messaging size and pending-call limits. Popup
routing supports all popup frames by default or the top-level frame with
`frameId: 0`; `documentId` and non-root explicit frame targeting remain
unavailable on this compatibility path.

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
  allows it in Extensions Manager. The private host receives Electron's native
  extension surface plus Summer's session-scoped runtime, action, cookie,
  ephemeral `storage.session`, private-tab, and declarative-net-request brokers.
  Those brokers authenticate the calling Electron session, route events,
  workers, and tab operations only within that session, and discard private
  action/session state with the private extension lifetime. Device-local
  `storage.sync`, managed storage, and regular-profile bookmark, history,
  download, window, and other browser-owned brokers remain unavailable rather
  than exposing durable profile state to a private extension context.
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
- `storage.sync` uses the local quota-compatible repository and optional
  [Mesh transport](extension-mesh-storage-sync.md). Enable Extension settings on
  both paired devices in My Devices. Only already active signed extensions
  participate; local/session data and extension permission grants are excluded.
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
Adblock Plus must expose enabled native static DNR rulesets, and Adblock Plus
must contribute its real **Block element** command to an image context menu. The
9GAG regression right-clicks one real post image, activates that command once,
and requires the resulting composer to contain a selected element, populated
filters, enabled Block and Preview controls, and the page overlay. This catches
message-delivery failures that still open an inert composer window.
The bundled API
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

The full Adblock Plus 9GAG regression is intentionally separate from the
menu-presence probe because it navigates to a live third-party site and activates
**Block element**. Run it explicitly from PowerShell with:

```powershell
$env:SUMMER_LIVE_EXTENSION_E2E = "cfhdojbkjhnklbpkdaibdccddilifddb"
$env:SUMMER_ADBLOCK_PLUS_9GAG_CONTEXT_MENU = "1"
npm run test:e2e:run -- playwright/popular-extensions-live.spec.ts -g "Adblock Plus opens an initialized Block element composer for a 9GAG image" --reporter=line
```

Set `SUMMER_LIVE_EXTENSION_DIRECTORY` as well to reuse an already extracted,
trusted Adblock Plus package instead of downloading it during the run.

### Upstream extension tests

#### Additional pinned suites

The following repositories are test-only submodules under `external/`. They
remain excluded from product inputs and release packaging. Each runner refuses
a dirty checkout or a revision different from its pin; do not edit upstream
assertions to obtain a passing result.

| Repository | Command | Actual scope |
| --- | --- | --- |
| `external/dark-reader` (MIT) | `npm run test:e2e:dark-reader` | Original MV3 Puppeteer/Jest browser suite, with Summer owning launch, installation, and new tabs. |
| `external/privacy-badger` (GPL-3.0-or-later, mixed component licenses) | `npm run test:e2e:privacy-badger` | Entire original in-extension QUnit suite in Summer. Other Python/Selenium functional scenarios are not included. |
| `external/vimium` (MIT) | `npm run test:e2e:vimium` | Original browser DOM suite in Summer, retaining upstream Chrome API stubs. Not installed-extension E2E or the separate Deno/JSDOM unit suite. |
| `external/violentmonkey` (MIT) | `npm run test:upstream:violentmonkey` | Original Jest/JSDOM suite with mocked extension APIs. This runs in Node, **not Summer**, and is not extension E2E coverage. |

Dark Reader's default selects every file in the pinned `tests/browser` tree.
Use `-- --file tests/browser/e2e/environment.tests.ts` for one file. Its first
run uses upstream's frozen `npm ci --ignore-scripts` and builds the original
MV3 test extension in an ignored stage. `-- --prepare-only` prepares without
launching; `-- --no-bootstrap` checks source/build and adapter-input hashes
before reuse. Original test-control sockets and fixtures bind only to loopback.
The browser test environment owns disposable Summer profiles and connects
Puppeteer to Summer's loopback debugging endpoint, not ChromeDriver. The two
upstream test-helper pages are still opened by the extension's own test build.

Privacy Badger and Vimium need no upstream dependency installation. They use
the checked-in original HTML runners and vendored test libraries. Both support
`-- --no-bootstrap` to reject a missing submodule. Privacy Badger also supports
`-- --keep-profile` for retaining only its disposable diagnostic profile.
`npm run test:e2e:privacy-badger:chrome -- --executable=<Chrome binary>` runs
the same pinned QUnit page in an explicitly selected Chrome-for-Testing binary
without downloading or modifying a browser installation. The binary must expose
browser-level CDP unpacked-extension installation and retain Manifest V2 support.
Vimium's DOM lane uses upstream Puppeteer's 800 by 600 default viewport, since
its offscreen-element assertions depend on viewport dimensions.

Violentmonkey requires Node 24 or newer (select with nvm). It copies clean
pinned source into a fresh ignored stage and uses a committed test-only npm
snapshot. Direct dependency versions and security overrides come from its
upstream lock; the frozen npm transitive resolution is not an identical pnpm
tree. No pnpm command or lifecycle script runs. The snapshot maintenance
procedure is in `scripts/violentmonkey-fixtures/README.md`. The root Summer
lockfile is unchanged.

Current Windows results for the new lanes: Dark Reader's complete MV3 browser
suite passed 29/29 tests across 12 files; Privacy Badger's original QUnit suite
passed 114/114 tests and 13,428/13,428 assertions with no skips; Vimium's
original DOM suite passed 109/109; and Violentmonkey's original mocked suite
passed 48/48 tests and 9/9 snapshots across seven files. Privacy Badger's first
Summer run was 96/114 and exposed four browser-compatibility defects: extension
API functions could not be stubbed, the Chrome 148+ `browser === chrome` alias
was absent, dynamic `scripting.registerContentScripts()` lifecycle methods were
missing, and callback-form errors also leaked a rejected Promise. The public API
surface is now replaceable like Chrome's, callback calls return `undefined`, and
the authenticated API broker implements bounded register/list/update/unregister
for MAIN and ISOLATED JavaScript content scripts. New registrations default to
ISOLATED; persisted MAIN registrations keep their world, and updates preserve
the current world unless explicitly changed. ISOLATED execution currently
requires an active managed MV3 extension with a service worker in the regular
profile. A bounded browser-to-worker request retains native scripting in the
extension's actual Chromium world, including its content-script runtime and
messaging. Each frame is matched separately; native document-ID translation
rechecks current host grants and the exact document before executing signed
package files. No synthetic world or page-world native bridge substitutes for
that route. An unavailable worker, private context, inherited-origin document
that cannot pass native document authorization, or stale navigation fails closed.
CSS registration remains unsupported, and `document_start` injection currently
runs at navigation commit rather than before page scripts. Electron does not attach session frame preloads to
MV2 `backgroundPage` WebContents, so Summer now stages the alias and dynamic
scripting relay before an MV2 extension's own background scripts; API permission
checks still execute in the main-process broker. Dynamic registration may
precede a host grant; each injection rechecks the current regular or private
profile grant, and committed uninstall purges the registration. MV2 rebroadcasts
preserve the authenticated content-frame sender after the main-process broker
resolves its exact registered target with the same native/public tab ID. The other
Privacy Badger Python/Selenium functional scenarios have not been run.

The available Chrome-for-Testing 147.0.7727.57 control ran the same 114 tests as
108 passed and 6 failed, all from `browser is not defined`; its API descriptors
confirmed writable/configurable Chrome methods but predate the Chrome 148
browser alias. Installed retail Chrome 152.0.7977.76 did not expose CDP
`Extensions.loadUnpacked`, so an equivalent current-Chrome unpacked control is
not claimed.

Dark Reader also exposed a page-CSSOM interference bug: Summer's website
drag-prevention rule was appended as an author `<style>` element. It now uses
native user-origin CSS, with the previous DOM protection as a failure fallback.
The dedicated website drag-region regression verifies that page and iframe
stylesheet counts are unchanged while author and inline `!important` drag rules
remain disabled, including after stylesheet removal and navigation. Existing
preload context exclusions and isolation are unchanged.

The real Privacy Badger manifest also exposed an installation and persistence
bug: its 391 content-script match entries exceeded a shared 256-item permission
budget. Summer now has a separate aggregate 4096-entry content-script-match
budget in both manifest inspection and stored-summary loading. This is a
Summer defensive bound, not a Chromium limit. The 256 API/host permission
limits, 512-character entry limit, 1 MiB manifest limit, and permission-review
checks remain unchanged. Regression tests cover the real manifest, overflow,
and persistence across manager restart.

`playwright/support/summer-extension-harness.ts` provides the reusable test
boundary for installing an arbitrary unpacked extension or Web Store package
through the real Extensions Manager review, then addressing its website tab,
action popup, extension pages, Manifest V3 worker, and diagnostics.

`playwright/upstream/adblock-plus.spec.ts` is the first upstream-derived
compatibility test. It independently implements the behavior of Adblock Plus's
GPL-3.0 `uses custom filters to block ads` smoke test at pinned upstream commit
`fc8af6fa168132de0b7198c5988e9aa7899e483b`; no upstream source or fixture is
copied into Summer. The first adapted slice serves two cosmetic targets and two
script-created network targets from a deterministic loopback fixture. It proves
all four load before installation, adds four fixture-specific filters through
Adblock Plus's Advanced options UI, then requires all four targets to disappear
while an unfiltered control remains visible.

Run it against the signed Web Store package with:

```powershell
$env:SUMMER_ADBLOCK_PLUS_UPSTREAM_E2E = "1"
npm run test:e2e:run -- playwright/upstream/adblock-plus.spec.ts --reporter=line
```

Set `SUMMER_ADBLOCK_PLUS_EXTENSION_DIRECTORY` to test a separately built,
unpacked Adblock Plus MV3 checkout instead. Summer's test-only
`external/adblock-plus` submodule pins the upstream source without vendoring it
into Summer's product code. The opt-in suite is not part of `npm run quality`
because its default signed-package path downloads from the live Web Store.

The initial 2026-09-02 signed-package probe installed Adblock Plus 4.43.4 and
reached its Manifest V3 worker, but incorrectly assumed its stock EasyList rules
contained the upstream test filters. The corrected deterministic port adds its
own filters, matching the upstream setup rather than treating that initial
failure as an ABP compatibility result.

#### Original Adblock Plus end-to-end suite

`playwright/upstream/adblock-plus-original.spec.ts` and
`scripts/run-adblock-plus-upstream-suite.mjs` run Adblock Plus's original Mocha
end-to-end tests directly from the test-only `external/adblock-plus` submodule.
The pinned source is commit
`fc8af6fa168132de0b7198c5988e9aa7899e483b` (Adblock Plus 4.43.3). Summer does
not vendor the GPL source into Summer's Git objects or package it in releases;
the parent repository records only the submodule commit. A unit gate forbids
product source and configuration from referring to `external/` except for the
explicit lint and packaging exclusions. Root ESLint and Electron Builder exclude
the directory from their inputs. The adapter attaches
ChromeDriver to a disposable Summer instance, preserves the upstream test hooks
and assertions, and translates only browser ownership operations such as
creating a real tab, closing disposable pages, reloading a worker, and applying
an unpacked upgrade through Summer's immutable-snapshot review flow. The
standalone uninstall case uses ABP's own loopback remote-config server and keeps
its production redirect inside the upstream mock proxy while preserving the
exact generated URL parameters asserted by the original test.

The pinned inventory contains 97 tests: 95 in the main suite, one custom
uninstall test, and one upgrade test. Of these, 89 apply to Summer's Chromium
runtime. The eight tests that do not apply are explicitly browser-specific in
the upstream source: one Firefox data-collection test, one Firefox Eyeometry
test, one Edge-only YouTube auto-allowlist test, one Firefox IPM test, and four
Firefox integrated-update campaign tests.

As of 2026-09-03, **88 of 89 applicable tests pass through Summer**, which is
also **88 of 88 locally runnable tests**. The remaining applicable Chrome
Eyeometry test is skipped by the upstream suite when `EYEOMETRY_BEARER` is not
available; it is not counted as passed. Results are aggregated from focused
suite shards because long single-process runs can hit Electron's open
[service-worker IPC deserialization crash](https://github.com/electron/electron/issues/52310);
the proposed upstream fix is
[Electron PR 52386](https://github.com/electron/electron/pull/52386).

The passing total is: 12 server/extension/ad-filtering smoke tests, 5
popup/Acceptable Ads tests, 13 filter-list tests, 8 data-collection/state/filter
experiment tests, 8 info-injector/snippet tests, 10 public-API tests, 6 premium
tests, 10 general IPM tests, 7 on-page-dialog tests, 5 new-tab campaign tests,
one Chromium update-campaign test, two uninstall tests, and one upgrade test.

Run all 88 locally runnable original tests, including the main, custom-uninstall,
and upgrade suites, with one command:

```powershell
npm run test:e2e:adblock-plus
```

The first run installs missing dependencies in the pinned submodule, builds its
Chrome MV3 development extension, downloads the newest released Chrome MV3
package older than the pinned source for the upgrade baseline, and downloads the
ChromeDriver matching Summer's castLabs Electron build when no matching explicit,
cached, temporary, or `PATH` driver exists. Later runs reuse downloaded inputs.
The wrapper packages the pinned current build for the upgrade step, verifies the
exact upstream commit, and sets the source, extension, driver, browser, and suite
environment automatically. Run one upstream main-suite Mocha shard with:

```powershell
npm run test:e2e:adblock-plus -- --suite main --grep "<suite or test title>"
```

The upstream and extension-directory environment variables remain available as
explicit overrides for a different checkout or build. `external/README.md`
records the submodule initialization and licensing boundary.

Pass `--suite main`, `--suite uninstall-custom`, or `--suite upgrade` to run one
entry point. The upgrade inputs are prepared automatically; both
`SUMMER_ADBLOCK_PLUS_CURRENT_BUILDS_DIRECTORY` and
`SUMMER_ADBLOCK_PLUS_UPGRADE_DIRECTORY` remain available as paired explicit
overrides. The upgrade reload is intentionally completed through the Extensions
Manager review rather than by mutating Summer's installed snapshot in place.
Supplying the upstream-owned `EYEOMETRY_BEARER` secret makes the final applicable
test runnable without putting that credential in Summer.

#### uBlock Origin Lite upstream filtering QA

`external/ublock-origin-lite` pins the official uBOL repository at commit
`72defaa1210a01f50b16f0e10f2bbb87cb990d7e`. Its GPL-3.0 source stays in a
separate submodule and outside Summer release artifacts. The test installs the
upstream repository's existing `chromium` extension through the real Summer
Extensions Manager, enables its `ubol-tests` static ruleset, and serves the
original `docs/tests` fixture directly from the submodule on loopback. No
upstream source is copied into Summer.

The suite covers the upstream unfiltered baseline plus optimal, complete,
basic, and no-filtering modes. Each scenario checks 45 Chromium/MV3 filtering
tiles and three exception tiles, for 48 applicable tile states per scenario and
240 tile-state expectations overall. The two fixture tiles explicitly marked
by upstream as Firefox MV2 uBO-only are excluded.

Run the complete pinned suite with:

```powershell
npm run test:e2e:ublock-origin-lite
```

The wrapper initializes the submodule when needed, verifies its exact commit and
rejects tracked or untracked changes in the upstream checkout, builds Summer,
and runs the test through Playwright's Electron connection. It does not use
ChromeDriver. Pass `-- --keep-profile` to preserve the disposable test profile
for diagnosis, or `-- --no-bootstrap` to fail when the submodule has not been
initialized.

#### Bitwarden upstream browser-interaction suite

The test-only `external/bitwarden-browser-interactions-testing` and
`external/bitwarden-clients` submodules pin Bitwarden's browser interaction
suite at `0e220a000c8c6818be66856aa38b63dfe1e46c87` and its extension source at
`b35a96bdc45a09f448ac200708c6ecdb5edacb9d`. Summer neither vendors those
GPL-3.0 repositories into its Git objects nor includes them in product or
release inputs.

`playwright/upstream/bitwarden.spec.ts` ports the five deterministic local
extension specs while importing the pinned upstream page inventory, selectors,
setup helpers, skip decisions, and expected values. The adapter changes browser
ownership only: the unpacked extension is installed through Summer's real
Extensions Manager, browser tabs are Summer tabs, and Playwright connects to
Electron instead of launching Chrome. The covered upstream specs are:

- `tests/static/autofill-forms.spec.ts`
- `tests/static/inline-menu.spec.ts`
- `tests/static/notifications.spec.ts`
- `tests/triggers/page-load-autofill.spec.ts`
- `tests/triggers/popup-autofill.spec.ts`

At the pinned revision the upstream inventory contains 72 applicable page
scenarios across 19 page definitions: 14 message-autofill cases, 12 inline-menu
cases, 11 page-load cases, 15 popup-autofill cases, and 20 notification cases.
Summer executes 69 of those page scenarios plus the page-load negative control.
The remaining three are the message, page-load, and popup variants for the
closed-shadow-root fixture. Those adapter skips were based on an earlier
assumption that `chrome.dom.openOrClosedShadowRoot` was unavailable. The direct
native content-world probe now confirms this API in the pinned runtime; the
three upstream Bitwarden scenarios have not been rerun by that probe, so their
adapter coverage remains unverified rather than proving an engine limitation.
Upstream's explicit
known-failure and not-applicable skips remain the authority. The live public-site
specs are intentionally outside this deterministic command, and the separate
web-client accessibility suite does not exercise a browser extension.

Run the full local extension suite with one command:

```powershell
npm run test:e2e:bitwarden
```

Docker Desktop must already be running. On first use the wrapper initializes
both shallow submodules, installs pinned npm dependencies, builds the Chromium
MV2 extension, binds its verified Chrome Web Store identity, prepares an
ignored local test account and certificate, and seeds the upstream cipher
inventory. Every run creates a loopback-only Bitwarden Lite Compose project and
removes its containers and database volumes in `finally`, including after a
test failure. Cached dependencies and revision/config/Web-Store-stamped build
outputs make later runs much faster without reusing an extension built from
different inputs. Run one adapter with, for example:

```powershell
npm run test:e2e:bitwarden -- --grep "original popup-autofill"
```

Pass `-- --keep-profile` to retain only the disposable Summer profile for
diagnosis. `-- --no-bootstrap` turns any missing submodule, dependency,
credential, certificate, or missing/stale extension build into an immediate
error. The runner never invokes ChromeDriver and never uses a real browser
profile.

The adapter must not use `collectPageDetailsImmediately` or inject an extra
`collectPageDetails` exchange as a readiness probe for UI-driven or page-load
scenarios: those messages can initialize the behavior under test. Explicit
message-autofill scenarios still send their intended upstream command. After
removing the initialization probes, the focused page-load run passed its ten
supported page cases and disabled-autofill negative control, but popup Fill
left the first login fixture empty. Repeating with upstream's `slowMo: 600`
also failed. That popup failure remains unresolved; the configured inventory
above is not a claim that every case currently passes.

#### MetaMask upstream E2E suites

`external/metamask-extension` pins MetaMask 13.48.0 at commit
`43f96f63129875db7ce7619250fc657fefee5ffc`. Its source-available license is
more restrictive than an OSI open-source license, so the checkout remains a
separate test-only submodule and is excluded from Summer's product, lint, and
release inputs.

At that revision MetaMask has two native Playwright extension specs, both under
`test/e2e/tests/settings/`, plus 354 files in its active default Mocha E2E lane:

- `terms-of-use.pw.spec.ts` loads an existing fixture wallet with an outdated
  terms timestamp, unlocks it, scrolls the update terms, accepts them, and
  requires the modal to close.
- `auto-lock.pw.spec.ts` unlocks the fixture wallet, selects the upstream
  quarter-minute auto-lock option through Settings, and requires the unlock
  page to return after the idle interval.

`playwright/upstream/metamask.spec.ts` preserves those scenario states,
selectors, actions, and assertions while changing browser ownership: the test
build is installed through Summer's real Extensions Manager and Playwright
drives Summer's Electron pages instead of launching Chrome. The fixture server
is loopback-only, each test uses a new disposable Summer profile, and no
recovery phrase is logged or attached.

Run the complete pinned native Playwright suite with one command:

```powershell
npm run test:e2e:metamask
```

Run all 354 original default-lane Mocha files through Summer with:

```powershell
npm run test:e2e:metamask:original
```

That command makes an ignored staged copy under `.tmp/`, installs only the
upstream runner's dependencies with frozen npm snapshots, and applies revision-checked adapter
overlays to MetaMask's incremental Playwright driver. MetaMask's specs, fixture
builders, page objects, and assertions remain unchanged. The adapter replaces
Chrome ownership with disposable Summer profiles and real Summer tabs and
extension dialogs. Its CSS-plus-text locator preserves upstream's exact XPath
alternatives, including removal of literal spaces, without weakening case or
whitespace matching. Local blockchain cases use MetaMask's checksum-verified,
pinned Anvil release. On Windows, Bitcoin cases automatically use Bitcoin Core
from `C:\Program Files\Bitcoin\daemon\bitcoind.exe` when it is installed; set
`SUMMER_METAMASK_BITCOIND_BINARY` to an absolute `bitcoind.exe` path to override
that location. On Windows, local Tron cases use Docker Desktop and automatically
run MetaMask's official `GreatVoyage-v4.8.1` java-tron image pinned by digest
`sha256:d51b3568c5b1b7d1fbc5cc97d1b266ef5b8c574dc62b08335eec398aeebd388a`.
Set `SUMMER_METAMASK_JAVA_TRON_DOCKER_IMAGE` to override that image reference.

The test-only `scripts/fixtures/metamask/npm-shrinkwrap.json` pins the npm
dependency graph. Ordinary reruns use `npm ci` with lifecycle scripts disabled;
prebuilt fixture archives have separate SHA-512 pins and are re-extracted from
verified bytes. Updating these snapshots is an explicit maintenance operation,
described in `scripts/fixtures/metamask/README.md`; it does not change Summer's
root lockfile. Node/npm still use the developer's configured nvm environment.
The MetaMask driver has an isolated, context-local text clipboard. Actual
application-produced copy values remain asserted, but these tests do not
exercise native OS clipboard permissions, non-text formats or trusted native
clipboard events. Driver-controlled main-world text copy/paste no longer reads
or replaces the user's clipboard; this is not an OS-level clipboard sandbox
for native context-menu actions or isolated content-script worlds.
Document-start callbacks retain their test references across MetaMask's
LavaMoat global scuttling, and only Playwright's injected transport properties
are protected; application security initialization remains enabled. The focused
real-DOM clipboard regression passes, but the original personal-sign scenario
still fails at its `Copied.` feedback checkpoint. A bounded diagnostic confirmed
that the actual copied fixture address reaches the context-local clipboard;
the browser-side completion acknowledgement remains unresolved. Do not treat
that scenario as passing or restore OS clipboard access to make it pass.

The first run initializes the shallow submodule and downloads the matching
official MetaMask Chrome test build from successful GitHub Actions run
`33788111146`. The wrapper verifies the 23.9 MB archive against SHA-256
`abb6ad38d67af1d5d00b3845d2967a47d100d7b13e1313711ef0e3f1407508fc`,
caps the response at 32 MB, and caches only those verified archive bytes. Each
command extracts a clean ignored `.tmp/` working copy so local modifications
cannot masquerade as the verified build. The official
test build intentionally uses MetaMask's test extension identity
`hebhblbkkdabgoldnojllkipeoacjioc`, distinct from the signed Web Store identity
covered by `playwright/popular-extensions-live.spec.ts`.

Run either native Playwright scenario by title, for example:

```powershell
npm run test:e2e:metamask -- --grep "automatically locks"
```

For a focused original Mocha file, pass its path exactly as it appears in the
MetaMask checkout. An optional `--grep` narrows that file further:

```powershell
npm run test:e2e:metamask -- --original-spec test/e2e/tests/dapp-interactions/provider-api.spec.ts
npm run test:e2e:metamask -- --original-spec test/e2e/tests/confirmations/signatures/signature-request.spec.ts --grep "can queue multiple Signature Requests of Sign Typed Data and confirm"
```

Pass `-- --keep-profile` only for diagnosis, or `-- --no-bootstrap` to require
that the pinned source and verified build cache already exist. The adapter
blocks non-loopback traffic so the test wallet cannot contact live backends.
Chrome and ChromeDriver are not used. The two native Playwright scenarios and
four focused original Mocha cases currently pass through Summer. The complete
354-file original lane is wired for one-command execution, but is not considered
passing until that command completes successfully.

#### React DevTools upstream Chrome launcher

`external/react-devtools` pins the React monorepo at
`8425b6915e8ebe2b12bf4bd42975bfccfa62fc5c` (React DevTools 7.0.1). React is
MIT-licensed, and the checkout remains a separate test-only submodule excluded
from Summer's product, lint, and release inputs.

At this revision the extension's Chrome test entry point,
`packages/react-devtools-extensions/chrome/test.js`, is a browser launcher rather
than an assertion suite. It requests the unpacked extension, an isolated profile,
automatic DevTools opening, and `https://react.dev/`. Summer executes that
unchanged entry point with a capture adapter for `chrome-launch`, validates the
captured launch contract, and translates it into a real Summer test:

```powershell
npm run test:e2e:react-devtools
```

The wrapper initializes and verifies the shallow submodule, then installs the
matching signed Web Store package with extension ID
`fmkadmapgofadopljbjfkapdkoienihi`. This launcher flow tests the signed release;
the separate inline suite below builds the pinned source with npm. The E2E requires the
installed version to match the pinned source manifest, starts the extension
worker, verifies the page-world React DevTools hook has a live Fiber renderer,
opens Summer's detached Chromium DevTools, and requires a React panel's real
`panel.html` to render a visible component-tree row without the "no React" fallback.
An empty panel cannot pass. Chrome and ChromeDriver are
not used.

This flow exposed a Summer-side preload regression: the general website preload
was also running in Chromium-owned, non-context-isolated DevTools content and
calling `contextBridge`, which throws there. Summer now rejects non-isolated,
`devtools:`, and `chrome-extension:` contexts before performing any website
preload side effect. Normal isolated website tabs keep the bridge, while the
Chromium-owned DevTools and extension contexts remain untouched.

The original inline browser suite is also available through Summer:

```powershell
npm run test:e2e:react-devtools:inline
```

The npm-only runner builds an ignored staging workspace and executes the nine
original Playwright scenarios with Summer in place of Playwright Chromium. By
default it follows React's compatibility workflow: current React plus 16.0,
16.5, 16.8, 17.0, and 18.0. This produces 54 outcomes; 46 pass and the eight
remaining cases are skipped by React's own version guards. The adapter waits
for the upstream fixture's React application and DevTools roots instead of
depending on browser timing; the readiness race reproduces in Chrome and does
not alter any functional assertion. Use `-- --current-only` for the nine current
React cases or `-- --react-version 18.0` for one compatibility target.

The first run downloads a checksum-verified Node 20.19.0 runtime (npm 10.8.2)
under `.tmp/react-devtools-runtime/`, without switching the developer's nvm
version. It uses `npm ci` with the test-only dependency snapshots under
`scripts/fixtures/react-devtools/`; normal reruns never resolve fresh dependency
versions. The source-build snapshot is generated from the pinned React
`yarn.lock` using npm's import support. Compatibility targets also have frozen
snapshots, including their transitive dependencies. No root or upstream lockfile
is changed.

The staging workspace at `.tmp/react-devtools-inline-stage/` caches the build.
Before reuse, the runner hashes source files and built outputs, including added
and removed files, and compares the runner/toolchain/dependency inputs. Modified
staged tests or builds invalidate the cache. `-- --no-bootstrap` fails on a
missing or stale runtime/stage instead of rebuilding. Both browser flows always
attempt disposable-instance cleanup, including when startup fails partway through.

When intentionally updating the pinned React revision or dependencies, regenerate
the snapshots with `node --use-system-ca scripts/update-react-devtools-dependencies.mjs`,
review the resulting test-only dependency changes, and rerun the browser matrix.

Pass `-- --url <https://example.test/>` to replace the default React page or
`-- --keep-profile` to retain the disposable profile for diagnosis. This proves
the upstream launcher contract and the matching signed release's functional
integration. Together, the launcher test and inline matrix cover React's
original browser-driven DevTools behavior.

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

## Web Authentication proxy preparation

`webAuthenticationProxy` is not supported by the installed Electron runtime.
Summer has prepared a browser-process native patch and authenticated page/worker
broker for its five methods and request events. Calls require a loaded MV3
extension with the required `webAuthenticationProxy` permission; the permission
cannot be optional. Missing native capabilities reject, and no renderer
replacement for `navigator.credentials` is installed. The preparation preserves
pending requests across worker suspension, cancels stale deliveries and keeps
native leases exclusively in main. Private execution remains unavailable.

The prepared remote-session notification hook uses the browser-owned user-data directory
and literal extension ID, with no file-content disclosure. It must not watch a
person's Chrome profile or synthesize session-state events. Native compilation
and disposable-profile runtime acceptance are still pending; an exposed API
object or passing broker fixture does not establish support. See
[the implementation checkpoint](chrome-extension-namespace-implementation.md#webauthenticationproxy-native-capability-gate)
and [the prepared native bridge](../native/electron/web-authentication-proxy/IMPLEMENTATION.md).

## Migrating extensions from another browser

Settings can discover extensions in conventional Chrome, Edge, Opera, Brave,
Vivaldi, and Chromium profiles. Discovery returns only a bounded display record
containing the validated Chrome Web Store ID, name, and version. Profile paths,
complete manifests, extension storage, cookies, account state, permissions, and
source-browser files never cross into the renderer.

Selecting an extension does not install or enable it. Summer hands the selected
IDs to the trusted Extensions Manager through short-lived, owner-, source-, and
profile-bound capabilities. The manager downloads each signed package again
from the Chrome Web Store and presents the normal access and compatibility
review one extension at a time. The user must choose **Add extension** for every
package. Cancelling one review skips only that extension; an extension already
managed by Summer remains in its current enabled or disabled state.

Before starting a desktop extension runtime, Summer durably records that exact
activation as incomplete. After runtime startup and initial content-script
synchronization succeed, the desktop host retains that marker for 30 seconds.
Successful observation, runtime unloading, or a normal browser quit clears it.
If Summer exits during this period, the next ordinary launch clears the stale
marker and retries the extension without changing the user's enabled choice.
An interrupted browser process is not evidence that an extension caused it;
browser crash recovery handles repeated startup failures and offers Safe Mode
without persistently disabling extensions. An extension that actually fails to
load is reported as a load failure while its enabled choice remains intact.

Older builds could automatically disable an extension after such an
interruption. Summer does not silently re-enable those previously disabled
records because an explicit Disable could have raced the earlier shutdown.
Instead it labels their recovery reason in Extensions Manager and offers a
manual Enable action. Packages, extension storage, and permissions are retained.

Summer deliberately does not copy enabled state, settings, local storage,
accounts, optional permissions, user-script access, file-URL access, or private
session access. Unpacked, component, enterprise-policy, private, and removed
Web Store extensions may be listed by the source profile but cannot be migrated
unless the official store can provide a correctly signed package. Arc is not
included until its extension profile layout is independently verified.

The scanner bounds directory enumeration and metadata reads, rejects symlink or
reparse-point escapes, resolves localized display names only inside the selected
extension version directory, and invalidates old snapshots when discovery is
refreshed. Tests must use disposable browser directories; never point automated
tests at a person's live browser profile.

## Desktop browsing-data storage removal

`browsingData.removeFileSystems`, `removeIndexedDB`, `removeLocalStorage` and
`removeServiceWorkers`, and the corresponding bulk `remove` categories, now
delegate to the regular Session's native Chromium data remover. Each request
selects exactly its requested data type. Origin inclusion or exclusion is passed
to that remover, including its native partitioned-storage matching; the browser
does not infer stored origins from open tabs or delete profile files directly.
`browsingData.removeCacheStorage` and `remove(options, {cacheStorage: true})`
clear Cache API data for explicit HTTP(S) `origins` in the regular session.
The stock runtime clears the first-party storage key for each selected origin.
Global, exclusion-only, and time-scoped requests are rejected. The operation
leaves partitioned third-party Cache Storage and the HTTP cache outside its scope.

These operations share the existing authenticated extension-page/worker broker
and reviewed `browsingData` permission. Private execution remains unavailable.

The installed Electron API does not expose a time boundary for these native
storage operations. A positive `since`, including a fractional millisecond, is
rejected before any deletion or saved-data confirmation. Empty origin inclusion
lists and simultaneous `origins`/`excludeOrigins` are rejected, and an explicitly
empty `originTypes` mask removes nothing. A required options object cannot be
omitted or replaced with null. Existing password/form-data confirmation remains
separate; this increment does not remove payment cards or extension storage.
`cacheStorage` remains separate unfinished work. The real Windows page/worker
fixture's four-storage-category campaign passed against two HTTP
origins with actual IndexedDB, local storage, OPFS files, CacheStorage sentinels
and service-worker registrations. It verifies each type's inclusion, exclusion,
global and no-op removal, rejection without mutation, unrelated-store/origin
survival, and extension storage survival. Promise/callback void results and
scoped callback errors are checked. Evidence is
`.tmp/bookmarks-browsing-data-live-evidence-parent-20261027.json`. Timed-scope
parity remains unfinished; these native storage calls cannot silently widen time
filters to all-time deletion.

### Retired browsing-data methods

`removeAppcache`, `removePluginData` and `removeWebSQL` complete as validated
no-ops. Their bulk keys and `serverBoundCertificates` are ignored after boolean
validation; a mixed request removes only its selected active categories. All
argument and active-scope validation completes before any active deletion or
saved-data confirmation. `cacheStorage` is active in Chrome and remains an
explicit unsupported request in Summer, rather than being treated as retired.

Retired-only requests accept finite negative/fractional `since` values and inert
protected-web/extension origin-type masks. Required object/boolean/list shapes,
empty inclusion and conflicting-filter checks still apply. Original origin
strings go to the regular Session's native `clearData` parser with a **literal
empty `dataTypes` list**. The pinned native implementation converts each string
through `GURL` and `url::Origin`, rejects opaque origins, then schedules no removal
operation for that zero mask. No arbitrary request options are forwarded. This
uses native file/blob/filesystem/registered-scheme validation without a JS origin
approximation or granting deletion of extension/protected data. Retired filters
are copied once, validated and frozen; they have no extra per-field size cap.
Existing transport limits are unchanged (the worker relay limits ordinary
requests to 1 MiB). Active mixed requests retain their existing filter-size,
HTTP(S), time and origin-type scopes.

The expanded page/worker fixture checks actual loopback OPFS, IndexedDB, local
storage, CacheStorage and service-worker sentinels, plus extension storage,
across retired calls and rejected mixed requests. The rebuilt Windows MV3
page/worker campaign passes, including native file/blob/filesystem/extension
origin handling, over-256 filter lists and long valid URLs. The scoped mixed
request removes only the selected site's local storage and preserves every
other sentinel. Evidence is
`.tmp/browsing-data-retired-live-evidence-parent-20261027.json`.
The earlier integrated 45-suite gate passed 1,058 tests, and all four selected
browser cases passed: storage/retired methods, bookmarks, portable services and regular
single-extension DNR. The multiple-extension/private DNR failure was excluded and
incomplete in that earlier campaign. Its original failed evidence is preserved;
the later cache-owner/static-settings acceptance is documented in the DNR section
above and `.tmp/dnr-full12-acceptance-parent-20261028.json`.
Those earlier results do not establish full type/release gates,
native engine compilation or untested platform/context parity.

Password and form-data behavior is unchanged. In pinned desktop Chromium 152,
`removePasswords` reports a deprecated error with its default feature enabled;
the bulk password key is ignored and settings marks it unselected. Summer still
offers confirmed all-time/profile-wide password deletion and advertises that
behavior. `formData` remains active in Chrome and includes timestamped form,
address and payment-related data; Summer's confirmed profile-only subset omits
payment cards and rejects unsupported scopes. These known differences require
separate product decisions, not silent changes under retired-method support.
