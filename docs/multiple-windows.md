# Multiple browser windows

Summer can open multiple regular or private browser windows inside one profile
process. Regular windows share profile data such as cookies, bookmarks,
settings, installed Summer Apps, and extensions. Each window owns its own
browser chrome, active tab, tab strip, overlays, widgets, notifications, and
native page views.

## Tab layouts

**Settings -> Appearance -> Tab layout** places the tab strip at the top,
bottom, physical left, or physical right of every regular and private browser
window. The **All sides** choice mirrors the same live tab strip on all four
edges; every copy can activate, close, keep, add, and reorder tabs. All copies
show the same window-owned tab order and active tab. Layout changes propagate
to open regular and private windows without broadcasting the full settings
store to private windows. Existing profiles default to the original top layout.

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
Closing the final tab in a window keeps that window open with a fresh new tab.
The [Panic button](./panic-browser.md) closes tabs across all windows in the
current profile and leaves one regular window with configured destinations.

## Hidden tab hierarchy

Each live tab belongs to an in-memory opener tree owned by the main process.
Tabs created from browser chrome are roots. A website tab opened from another
tab is a child of that opener and is inserted immediately beside it. The tree
has no visual treatment yet; it is foundation for later tab-group features.

Dragging a tab in any visible strip moves that tab and all of its descendants
as a single ordered block. Dragging a child also detaches that subtree from its
former parent, making the dragged tab a root. Closing a parent promotes its
direct children to the closed tab's parent so no live tab is orphaned. Moving a
single tab to another browser window also makes it a root because hierarchy
edges never cross windows.

The hierarchy survives a browser-chrome renderer reload because the main
process sends it back with the window tab snapshot. A parent edge is also
written to disk when both the parent and child are kept tabs. If only the child
is kept, it restores as a root; this prevents a kept tab from depending on a
tab that will not return. Crash-only restored tabs and private hierarchy data
remain outside this durable hierarchy.

Every browser-chrome surface and tab also binds its `WebContents` to that same
`BaseWindow` through Electron's native owner relay. Moving a tab clears the
previous native owner before binding the destination. Browser-owned message
boxes explicitly restore focus to the requesting chrome after they settle.
This keeps download, JavaScript, file, and other native dialogs attached to the
visible Summer window instead of leaving a parentless modal that can make
browser chrome appear unresponsive.

On Windows, Electron's native window-controls overlay remains right-aligned
even for RTL interface locales. Summer therefore disables that overlay for RTL
managed browser windows and renders close, maximize/restore, and minimize
controls on the physical left. The controls call only their authenticated
owner window and leave the title bar during fullscreen. LTR Windows and other
platforms retain their native controls. Windows system shortcuts, including
`Alt+F4` and `Win+Z`, remain available independently of the browser renderer.
If the RTL browser chrome stops responding or crashes, the main process shows
an owner-scoped native recovery dialog for reload or close; a bounded reload
attempt re-prompts if the controls do not recover.
Website-owned popup windows also retain native caption controls so untrusted
web content cannot replace the only close, maximize, or minimize affordances.

## Persistence

Kept tabs from every regular window are persisted in deterministic
primary-window-first order. On the next process launch they restore into the
primary regular window. Summer does not currently restore the previous native
window count, placement, or geometry. Private windows and private tabs are
never persisted.

After an unclean process exit, Summer also offers to restore a bounded URL-only
snapshot of all live regular tabs into the primary window. The user can restore,
start fresh, or quit before any browser window is created. The previous window
count and geometry are intentionally not recreated. See [Crash recovery](./crash-recovery.md).

## Trust boundary

Renderer requests never choose an Electron window ID. The main process derives
the owning `BrowserWindowContext` from the authenticated top-level browser
chrome sender and rejects cross-window tab operations. Profile-global updates
are broadcast to live regular chrome renderers, while window-local surfaces are
routed only to their owner.

## Verification

Use disposable `--instance-id` and `--instance-data-dir` values. Focused checks:

```sh
npm run test:unit -- tests/regularWindows.test.ts tests/multiWindowTabPreservation.test.ts tests/windowLifecycle.test.ts tests/notificationActions.test.ts tests/extensionSidePanels.test.ts tests/nativeWindowOwnership.test.ts tests/browserWindowIpc.test.ts
npm run test:e2e:run -- playwright/windows/multiple-windows.spec.ts
```
