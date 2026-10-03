# Private browsing (incognito)

Private browsing opens a second, tabbed browser window that shares Summer's Vue
interface but does not write browsing state into the regular user profile: no
cookies, cache, history, session, downloads list, preserved tabs, saved
passwords, autofill profiles, or credit cards are merged into that profile. It
is a separate disposable runtime surface, not another durable user profile.

On phones, private tabs live alongside regular tabs in the mobile tab switcher.
They follow the same privacy outcome through platform-native isolation rather
than an Electron window.

## Mobile private tabs

- **New private tab** is available from the phone navigator and Tabs library.
  The active private tab carries a persistent private badge and distinct chrome
  treatment so it cannot be mistaken for a regular tab.
- Private URLs, titles, active state, and back/forward trails never enter the
  logical mobile tab session. Relaunch restores only regular tabs. Closing the
  final private tab returns to a regular survivor, or creates a new regular
  blank tab when none remains.
- Popups and explicit open-in-new-tab actions inherit the source tab's mode.
  Operating-system deep links always open in regular browsing so an external
  application cannot silently enter or disclose a private lifetime.
- iOS private tabs share one `WKWebsiteDataStore.nonPersistent()` store for the
  current private lifetime. It is released after the last private tab closes.
- Android private tabs require the installed WebView provider's AndroidX
  `MULTI_PROFILE` capability. Summer assigns a randomly named profile before
  configuring or navigating each private WebView, shares it among live private
  tabs, destroys the views before deleting the profile, and never reuses that
  profile name. Private creation fails closed when the provider cannot supply
  this boundary; clearing the regular cookie store is never used as a fallback.
- The initial mobile policy disables extensions, remembered website permission
  decisions, and website notifications in private tabs. Android private tabs can
  export HTTP(S) downloads to a document explicitly selected in the system picker.
  The transfer uses that private WebView profile's cookies, keeps no Summer
  download record or notification, and stops when the tab closes. Redirects
  stay on the exact scheme, host, and port, so private cookies never follow a
  cross-origin redirect. The complete response is held in bounded private
  memory before Summer writes to the selected document; a failed network
  transfer cannot leave a partial file. Same-document `blob:` and bounded
  `data:` downloads use the same document picker and private export path,
  with the live page identity checked before every generated chunk. These
  exports run one at a time and are limited to the lesser of 64 MiB or one eighth
  of the device's Java heap (with an 8 MiB floor). Queued exports are deleted
  if the tab or browser closes before they start. The user-selected file remains outside Summer's
  private profile after the private session ends. Android
  keeps OS autofill disabled for private views, but lets a user explicitly fill
  an existing Summer-vault password after device authentication. Private form
  submissions never offer to save or update the durable vault. iOS keeps website
  data in the non-persistent store, but public WebKit API does not let Summer promise that
  the operating system will never offer its own credential UI. Bookmarks and
  durable Summer App/provider actions are not created from a private page. These
  restrictions avoid routing private activity into stores that do not yet have
  a separately reviewed private lifetime.

Native source and contract tests verify these boundaries, but they do not prove
an installed WebView or WebKit runtime. Release acceptance still requires live
Android and iOS tests showing that regular data is invisible in private tabs,
live private tabs share only their disposable store, a later private lifetime is
empty, renderer replacement stays private, and process termination never
restores a private URL.

## Opening and closing

- **Keyboard shortcut `Cmd+Shift+N`** (macOS) / `Ctrl+Shift+N` (other
  platforms). `createIncognitoWindow()` in
  `electron/main/execution/initApp.ts` builds a new `BaseWindow`, wires it
  through `configureBrowserWindow(baseWindow, {incognito: true})`, and mounts the
  same `BrowserWindowUI` used by a normal window. The window hides the native
  menu bar, so this accelerator is the primary entry point.
- The window title uses `app.window.incognitoTitle`, and the tab strip shows the
  `tabs.incognito.badge` eye-slash indicator. The profile switcher is hidden so a
  private session can never be confused with a profile.
- Closing the final private tab immediately opens and focuses a fresh private
  new tab, so the private window remains usable without crossing session bounds.
- Closing the window disposes its notification host and calls
  `tabs.closeAllForWindow(baseWindow)`; kept-tab persistence is skipped because
  the session must leave nothing behind.

## Isolation model

- Each Summer process owns one randomly named private-session directory beneath
  the operating-system temporary directory. Electron opens it through
  `session.fromPath(..., {cache: false})`; it never shares the regular Summer
  profile. This path-backed session is required because Electron can load
  extensions only into persistent sessions.
- Summer records the one directory it creates, unloads private extensions,
  calls Electron's data-clear and connection-close operations, and attempts to
  remove the entire directory after the final private window closes. Either a
  complete Chromium clear or complete physical removal must succeed before a
  later private lifetime can start. Windows can retain locks on cleared Chromium
  database sidecars until process exit; those files are retried on later
  lifetimes and startup. The isolated path has an owner marker bound to both the
  process ID and operating-system process-start identity. Startup removes
  recognized leftovers whose owner identity is no longer live, while leaving
  live parallel Summer instances and unknown entries alone.
- Built-in `summer-internal`, `summer`, and `sap` protocol handlers and the
  download session listener are registered on the incognito session too, so
  Summer App pages work identically but remain isolated from the regular
  profile and are included in private-lifetime cleanup.
- Allowed Chrome extensions use private-session workers, cookies, ephemeral
  `storage.session`, private-window tab operations, declarative network rules,
  and toolbar action state. Private tab queries and events observe only private
  windows. Summer's API relay rejects `storage.sync`, managed storage, and other
  private calls to brokers that still own durable regular-profile data instead
  of silently routing those calls through the regular session.
- Permission decisions (`setPermissionCheckHandler` /
  `setPermissionRequestHandler`) run through the same
  `PermissionsRuntime` with `persistAllowed = false`; allow/deny choices are
  never written to disk.
- The partition has the same user agent as the normal session.

## What incognito never touches

- Widgets: private windows do not restore saved widgets or offer browser-owned
  or Summer App widgets. Explicit opens and app host-open commands are blocked,
  including Carver previews and sandboxed widget frames. The renderer waits for
  the window context before reading widget layout storage, so opening a private
  window cannot migrate or overwrite the regular layout. Ordinary browser
  accessibility controls remain available independently of widget surfaces.
- Saved passwords, autofill profiles, and credit cards: IPC handlers in
  `passwordmanager.ts`, `autofill.ts`, and `creditCardManager.ts` return empty or
  reject before reading or writing when `isIncognitoWebContents` matches.
- Tab preservation (`tabpreservation.ts`): incognito tabs are never preserved,
  restored, or auto-restored; the preserved-tabs index and renderer update are
  always sent to the primary window's chrome.
- Whole-process crash recovery (`crashSessionRecovery.ts`) snapshots only the
  portable regular-tab projection. Incognito URLs, titles, active-tab state,
  and navigation history never enter `CrashSessionState.json` and are never
  offered after relaunch.
- Session restore (`sessionrestore.ts`), visit order (`tabOrder.ts`), and the
  tab restore service (`tabrestoreservice.ts`) skip incognito windows and tabs,
  so private activity can never appear after a restart.
- Frequent-site Keep suggestions (`frequentUrlKeepSuggestion.ts`) reject
  incognito tabs before URL normalization or store access. Private navigation
  therefore cannot increment, create, defer, or complete a suggestion record.

## Window management

`electron/main/front/windows.ts` keeps a `windowManagement` registry of every
browser window with its incognito flag. `mainWindow`/`mainBaseWindow` globals
remain compatibility aliases for the focused window; sender-driven operations
resolve the exact owning context instead. `getPrimaryContext()` returns the
first non-incognito window for startup-only work. Regular and private windows
keep independent tabs, overlays, notifications, and native page views. An Open
in New Window action from private chrome opens another private window rather
than moving its destination into the persistent session.

## Security notes

- Incognito windows are browser-owned UI like any other window; web content still
  runs under the same sandboxing and navigation restrictions.
- Private browsing is not a profile, an account, or a VPN, and it does not hide
  activity from the network or the site you visit.
- Extension access is disabled separately for every extension by default. A
  user can opt one in from Extensions Manager. Allowed extensions are loaded
  into the isolated private session, not the regular session's storage. Summer
  refreshes already-open matching private pages when access is granted, and
  routes toolbar actions and extension popups through that same private session.
  Summer refuses extensions whose manifest declares `"incognito": "not_allowed"` and
  reports that its private host always uses split, isolated execution.
- Native popups opened by private pages close with their opener, ensuring no
  unleased popup can retain the private session after its owning window closes.
- Private storage deletion is best effort, not a secure-erasure guarantee.
  While a private window is open, Chromium may write into the temporary
  directory. A crash, forced shutdown, filesystem snapshot, backup, malware, or
  forensic recovery can leave or recover those bytes until startup cleanup (or
  the operating system) removes them. Use full-disk encryption where recovery
  of temporary files is a concern.
