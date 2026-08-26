# Private browsing (incognito)

Private browsing opens a second, tabbed browser window that shares Summer's Vue
interface but does not write browsing state into the regular user profile: no
cookies, cache, history, session, downloads list, preserved tabs, saved
passwords, autofill profiles, or credit cards are merged into that profile. It
is a separate disposable runtime surface, not another durable user profile.

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
- Permission decisions (`setPermissionCheckHandler` /
  `setPermissionRequestHandler`) run through the same
  `PermissionsRuntime` with `persistAllowed = false`; allow/deny choices are
  never written to disk.
- The partition has the same user agent as the normal session.

## What incognito never touches

- Saved passwords, autofill profiles, and credit cards: IPC handlers in
  `passwordmanager.ts`, `autofill.ts`, and `creditCardManager.ts` return empty or
  reject before reading or writing when `isIncognitoWebContents` matches.
- Tab preservation (`tabpreservation.ts`): incognito tabs are never preserved,
  restored, or auto-restored; the preserved-tabs index and renderer update are
  always sent to the primary window's chrome.
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
  refuses extensions whose manifest declares `"incognito": "not_allowed"` and
  reports that its private host always uses split, isolated execution.
- Native popups opened by private pages close with their opener, ensuring no
  unleased popup can retain the private session after its owning window closes.
- Private storage deletion is best effort, not a secure-erasure guarantee.
  While a private window is open, Chromium may write into the temporary
  directory. A crash, forced shutdown, filesystem snapshot, backup, malware, or
  forensic recovery can leave or recover those bytes until startup cleanup (or
  the operating system) removes them. Use full-disk encryption where recovery
  of temporary files is a concern.
