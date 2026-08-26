# Opening local files

Summer accepts operating-system file drops on the browser controls or normal
web content. The browser
first displays a review sheet with a per-file recommendation; dropping a file
never launches it by itself.

## Routing

Summer resolves recommended apps from each enabled, desktop-compatible app's
validated `fileHandlers` manifest declarations. A unique match is shown by app
name. If several apps claim the same extension, Summer shows the candidates but
does not choose by installation order; the recommended action fails closed and
the user can use the browser or an operating-system app instead.

This is Summer's internal routing contract. It does not register the app as an
operating-system file association; packaged OS associations remain a separate
browser-level installation concern.

- Markdown (`.md`, `.markdown`), JSON (`.json`), and text (`.txt`, `.log`) open
  in the built-in Text Workspace. `.jsonc` opens in text mode so comments are
  not silently rejected by the JSON editor.
- Supported local video formats open in `sap://videoplayer/`.
- PDFs, images, and audio formats that Chromium can safely display open in a
  normal Summer tab.
- Unsupported files may be handed to the operating system after a separate
  confirmation. Executables, scripts, shortcuts, and active-content formats
  remain blocked.

The editable-file limit is 4 MiB and a single drop is limited to 32 files.
Text Workspace accepts UTF-8 text, preserves a UTF-8 byte-order mark, validates
JSON before saving, and refuses to overwrite a file that changed after it was
opened.

## Security boundary

Browser-chrome and ordinary-page preloads convert user-dropped `File` objects to native paths
with Electron's `webUtils.getPathForFile`. Only the privileged main process
receives those paths. It resolves and checks each regular file, retains the
paths in a short-lived session, and returns path-free descriptors to the Vue
renderer. The ordinary-page preload captures the gesture internally but exposes
no file API or path to the website; Summer Apps keep their own intentional drop
behavior. Website upload zones keep precedence when they accept the drop; the
browser review handles only an otherwise-unclaimed file drop.

The selected Summer App receives an opaque grant bound to its exact app,
handler, tab, and window through `window.summerFile`. It receives safe metadata,
a `summer-file://` read URL, and a revision—never a path or general
filesystem/IPC capability. Only an `edit` handler can save, and saves are capped
at 4 MiB. Summer opens and revalidates the granted file immediately before a
durable same-file write, preserving its existing filesystem identity and
security metadata instead of replacing it with a newly created file. If that
write fails after mutation starts, Summer makes a best-effort synced restore of
the original bytes. Grants expire and are revoked when their tab closes.
Text Workspace remains offline-only and its Markdown preview escapes raw HTML
and blocks image loading.

Generated app output is refreshed with:

```sh
npm run build:sap -- text-workspace
```
