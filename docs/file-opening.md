# Opening local files

Summer accepts operating-system file drops on the browser controls or normal
web content. The browser
first displays a review sheet with a per-file recommendation; dropping a file
never launches it by itself.

## Routing

Summer resolves recommended apps from each enabled, desktop-compatible app's
validated `fileHandlers` manifest declarations. A unique match is shown by app
name. If several apps claim the same extension, Open recommended asks the user
which matching Summer App to use instead of choosing by installation order.
The choice is reused for other files with the same extension and exact candidate
set in that batch, so a multi-file drop does not repeat the same prompt. Candidate
metadata is capped at 16 apps; if more match, the selector offers the explicit
system-app route rather than returning an unbounded manifest-derived dialog.
Cancelling that selector opens nothing. If a trusted File Explorer request chose
a Summer handler that then became unavailable, Summer tries the operating-system
route; identity or policy validation failures always fail closed without that
fallback.

This is Summer's internal routing contract. It does not register the app as an
operating-system file association; packaged OS associations remain a separate
browser-level installation concern.
The packaged browser declares a fixed set of built-in formats as operating-system
handlers; see [default browser integration](default-browser.md). Choosing Summer
for one of those formats in the operating system is an explicit file-open
request. It does not change the separate drag-and-drop review policy.

- Markdown (`.md`, `.markdown`), JSON (`.json`), and plain text (`.txt`, `.log`)
  open in the built-in Text Workspace. `.jsonc` opens in text mode so comments
  are not silently rejected by the JSON editor.
- Supported local video formats (`.mp4`, `.m4v`, `.webm`, `.ogv`, `.ogm`,
  `.ogg`, `.mov`, `.mkv`, `.mk3d`, `.3gp`, and `.3gpp`) open in
  `sap://videoplayer/`. Container routing does not guarantee that Electron can
  decode every codec track inside a file.
- Word `.docx`, Excel `.xlsx`, and PowerPoint `.pptx` files open read-only in
  `sap://office-viewer/`. The bundled viewer renders the file locally and never
  uploads it. Word layout, spreadsheet formatting/formulas, and PowerPoint
  transitions may differ from Microsoft Office. Legacy `.doc`, `.xls`, `.ppt`
  and macro-enabled Office formats do not have a Summer viewer handler.
- Netron can be installed from the bundled Apps catalog while offline. After
  installation, its read-only handler offers common model formats such as
  `.onnx`, `.tflite`, `.gguf`, and `.safetensors`. Until then, it does not claim
  model files.
- PDFs, images, and audio formats that Chromium can safely display open in a
  normal Summer tab.
- `Open here` on a trusted Files search result delegates an otherwise unhandled
  extension (including no extension) to the operating system's registered
  handler. On Windows, Summer opens the system app selector if no default
  association succeeds. XML uses this system route rather than being rendered
  as browser content. Executables, scripts, shortcuts, and other active-content
  formats remain blocked.

System delegation first invokes the registered desktop handler. On Windows, a
missing or failed association opens the operating system's app selector; Summer
reports that the chooser was shown without claiming that the user selected an
app. macOS uses its application chooser and waits for selection or cancellation.
Electron exposes no portable Linux application-selector API, so Linux reports a
missing association instead of pretending the file opened. This
same default-or-selector route is used by File Explorer's explicit system-open
action. Files actions return bounded path-free outcome categories for stale
tokens, changed or unavailable files, policy blocks, missing Summer/system
handlers, and cancellation. A failed action is reported visibly instead of being
silently discarded.

The editable-file limit is 4 MiB and a single drop is limited to 32 files.
Text Workspace accepts UTF-8 text, preserves a UTF-8 byte-order mark, validates
JSON before saving, and refuses to overwrite a file that changed after it was
opened.

## Security boundary

Browser-chrome and ordinary-page preloads, including eligible embedded frames,
convert user-dropped `File` objects to native paths with Electron's
`webUtils.getPathForFile`. Only the privileged main process
receives those paths. It resolves and checks each regular file, retains the
paths in a short-lived session, and returns path-free descriptors to the Vue
renderer. The ordinary-page preload captures the gesture internally but exposes
no file API or path to the website; Summer Apps keep their own intentional drop
behavior. Website upload zones keep precedence when they accept the drop; the
browser review handles only an otherwise-unclaimed file drop.

Both browser controls and ordinary-page preloads use `shared/fileDrop.ts` to
exclude HTML content drags from this fallback. Chromium can include a virtual
`File` when dragging a webpage image; moving that image must not open a disk-file
review or report that it has no native path. File-only drops (including native
file-manager URI lists) remain eligible, and website upload handlers retain
precedence.

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
Office Viewer is also offline-only. It refuses compressed files over 30 MiB,
archives with more than 2,000 entries or 160 MiB of actual expanded content,
and any single entry over 40 MiB. Converted PowerPoint HTML is displayed in an
opaque-origin sandboxed frame with a restrictive content policy; the app never
grants write access to Office files. Word HTML altChunks are disabled, and the
app policy blocks external resources.
Netron Model Viewer is an optional, offline-only, read-only Summer App for
machine-learning models. It registers common single-file model formats for
Summer's reviewed file opening; it is not installed by default. A granted file
has a 256 MiB limit and cannot expose sibling files. Formats with related
files can instead be selected together through Netron's own file picker after
the app is installed. The bundled Netron web build has startup telemetry,
remote model loading, and its age-based update gate disabled. Its app policy
allows local resources and the one granted `summer-file://` read only.

Generated app output is refreshed with:

```sh
npm run build:sap -- text-workspace
npm run build:sap -- office-viewer
npm run build:sap -- netron-viewer
```
