# Multiple browser windows

Summer can open multiple regular or private browser windows inside one profile
process. Regular windows share profile data such as cookies, bookmarks,
settings, installed Summer Apps, and extensions. Each window owns its own
browser chrome, active tab, tab strip, overlays, widgets, notifications, and
native page views.

## Opening windows

- **New Window:** `Cmd+N` on macOS or `Ctrl+N` on Windows and Linux, or File ->
  New Window.
- **Open in New Window:** a supported bookmark, tab suggestion, Summer App, or
  URL suggestion can open directly in a new window.
- **New Incognito Window:** `Cmd+Shift+N` on macOS or `Ctrl+Shift+N` elsewhere.
  An Open in New Window request made from private browser chrome stays private
  and opens another incognito window.

Closing one window closes only the tabs and native views owned by that window.
The profile process remains alive while another browser window is open.

## Persistence

Kept tabs from every regular window are persisted in deterministic
primary-window-first order. On the next process launch they restore into the
primary regular window. Summer does not currently restore the previous native
window count, placement, or geometry. Private windows and private tabs are
never persisted.

## Trust boundary

Renderer requests never choose an Electron window ID. The main process derives
the owning `BrowserWindowContext` from the authenticated top-level browser
chrome sender and rejects cross-window tab operations. Profile-global updates
are broadcast to live regular chrome renderers, while window-local surfaces are
routed only to their owner.

## Verification

Use disposable `--instance-id` and `--instance-data-dir` values. Focused checks:

```sh
npm run test:unit -- tests/regularWindows.test.ts tests/multiWindowTabPreservation.test.ts tests/windowLifecycle.test.ts tests/notificationActions.test.ts tests/extensionSidePanels.test.ts
npm run test:e2e:run -- playwright/windows/multiple-windows.spec.ts
```
