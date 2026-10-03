# Default browser integration

Summer's General settings show the operating system's current HTTP and HTTPS
handlers. Summer reports **Default** only when the operating system confirms that
both schemes open in Summer. The card refreshes whenever Settings regains focus,
so a choice completed in system settings is reflected without restarting the
browser.

## Windows

A packaged Windows build registers Summer as a handler for HTTP and HTTPS in its
AppX manifest. The action in General settings opens Windows Default Apps, targeted
to the installed Summer package when its AUMID is available. Windows protects the
final default-app choice behind its
system UI, so Summer does not write default-association registry values or claim
success based on Electron's AppX registration return value. The user confirms
Summer for HTTP and HTTPS in Windows, returns to Summer, and the live status
compares the selected handlers' executable paths with the running Summer build.

An unpackaged development instance deliberately cannot register itself as the
machine's browser. This prevents local Electron test executables from replacing
an installed browser.

The shared protocols entry in electron-builder.json5 registers HTTP and HTTPS.
Electron Builder combines that list with the Windows-specific protocols list,
which contains summer-shortcut and mailto, so a scheme must not appear in both
places. The packaged app can therefore appear as an email-link handler in
Windows Default Apps without silently replacing the user's current choice.
When selected, cold-start and second-instance mailto activations use Summer's
saved-provider compose routing. If no reviewed provider route is available,
Summer reports that safely instead of handing the URI back to itself. The AppX
maxVersionTested value also stays new enough for supported Windows 11 builds to
index Summer for the targeted Default Apps URI. Older supported Windows builds
may open the generic Default Apps page instead.

The package also offers Summer for HTML-family pages (.html, .htm, .xhtml,
.xht, .shtml, .shtm), PDFs, supported images and audio, local video, text and
Markdown, JSON, and the read-only Office formats .docx, .xlsx, and .pptx.
Each file type remains the user's choice in Windows Default Apps or Open With;
installing Summer does not replace its current handler. HTML, PDF, images, and
audio open in normal browser tabs. Video, text, and Office files open through
their built-in Summer Apps and scoped file grants. An unavailable Summer App
does not hand an associated file back to the operating system, which could
launch Summer repeatedly. Static local .shtml pages do not process server-side
includes. Optional catalog apps such as Netron are not claimed by the installer.

Summer also recognizes `claude://` links from websites as links to the installed
Claude Desktop app. A website must receive Summer's transient external-app
permission before the link is handed to the operating system. Summer does not
register itself as a handler for that scheme, and its mail compose routing is
limited to `mailto:` links.

## macOS and Linux

The existing direct Electron registration path remains in place. Summer rereads
the operating-system state after the request and reports success only when both
web schemes are confirmed. macOS can fall back to its Default Web Browser
settings page if the direct request is not accepted.

The packaged macOS app offers the same file types through its document-type
declarations. The Linux AppImage does not install desktop file associations by
itself; desktop integration is controlled by the user's AppImage tooling.

## Verification

The pure registration and status rules are covered by
tests/defaultBrowser.test.ts. Windows packaging coverage in
tests/appxManifest.test.ts prevents HTTP, HTTPS, or mailto registration from
being dropped. Launch-argument and mailto wiring tests cover normalized
activation and the no-relaunch fallback. The Settings card has an Electron
end-to-end test that replaces only the two default-browser IPC handlers and
verifies the visible state transition without changing the developer machine's
real default browser.

File association tests compare the installer list with the launch allowlist and
the built-in App manifests. A Windows package inspection and a real OS
selection/open test are still needed for release validation.