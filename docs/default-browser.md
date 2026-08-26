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

## macOS and Linux

The existing direct Electron registration path remains in place. Summer rereads
the operating-system state after the request and reports success only when both
web schemes are confirmed. macOS can fall back to its Default Web Browser
settings page if the direct request is not accepted.

## Verification

The pure registration and status rules are covered by
tests/defaultBrowser.test.ts. Windows packaging coverage in
tests/appxManifest.test.ts prevents HTTP, HTTPS, or mailto registration from
being dropped. Launch-argument and mailto wiring tests cover normalized
activation and the no-relaunch fallback. The Settings card has an Electron
end-to-end test that replaces only the two default-browser IPC handlers and
verifies the visible state transition without changing the developer machine's
real default browser.
