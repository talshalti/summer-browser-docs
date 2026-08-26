# Private browsing (incognito)

Private browsing opens a second, tabbed browser window that shares Summer's Vue
interface but keeps no trace in the user profile: no cookies, cache, history,
session, downloads list, preserved tabs, saved passwords, autofill profiles, or
credit cards. It is a separate runtime browser surface, not a profile.

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

- Every incognito tab renders on the in-memory session
  `INCOGNITO_PARTITION = "incognito"` created with `{cache: false}` in
  `electron/main/front/sessions.ts`. There is no `persist:` prefix, so Electron
  never writes an on-disk partition for it.
- Built-in `summer-internal`, `summer`, and `sap` protocol handlers and the
  download session listener are registered on the incognito session too, so
  Summer App pages work identically but stay in-memory.
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
- No extension incognito-splitting is provided; extensions are not granted
  private-session access (see `docs/chrome-extensions.md`).
