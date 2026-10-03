# Summer Browser privacy policy

Last reviewed for Summer 2.10.0.

Summer Browser does not operate advertising services and does not sell or share
data for advertising. Summer 2.10.0 does not collect product usage events, AI
request telemetry, AI task descriptions, or AI request text. The new telemetry
and mandatory agreement remain inactive, including for profiles with saved
acceptance preferences from development builds.

Existing privacy-filtered technical crash reporting remains available under its
own preference, described below. The private mode control at `summer://telemetry`
cuts communication with Summer-operated servers, including crash-report uploads
and the default Mesh coordinator.

In supported packaged desktop releases, privacy-filtered technical crash reporting is
enabled by default. The user can turn it off in About, or with private mode. When Summer detects a supported
crash incident, it automatically queues a narrowly scoped report and sends it
to the Summer-operated collector. Recovery choices and clearing the general
diagnostics log do not change this preference. Turning reporting off deletes unsent reports and prevents future reports, but cannot retract a report already accepted by the collector. Summer does not send
browsing history, URLs, tab titles, page or form contents, credentials, autofill
or payment-card records, cookies, tokens, clipboard data, prompts, profile
identity, or private-window state to the crash collector.

## What this policy covers

This policy covers data collection performed by Summer Browser and services
operated by the Summer Developer Team. Browser profile data is kept on the
user's device unless the user deliberately exports or shares it.

## Usage and AI telemetry

The new usage and AI telemetry implementation is disabled in Summer 2.10.0 by
a source-owned release switch. Summer does not record feature counts, startup
performance events, AI request outcomes, task labels, or request text for this
telemetry. It does not create a telemetry installation identifier or contact
`https://telemetry.summerbrowser.com` to upload events or fetch improvements.
Preferences, environment variables, and browser-page controls cannot enable
this collection. The browser does not show the proposed mandatory Terms of Use
agreement during setup or when existing profiles start.

AI providers and apps chosen by the user may still process requests to perform
the requested AI features under their own data-handling policies. Disabling
Summer's new telemetry does not change those requested features or existing
technical crash reporting. Any future release that enables telemetry must first
publish its reviewed data-handling policy and finalized agreement.

## Local diagnostics

Summer keeps a small rotating diagnostic log inside each active browser
profile. It records bounded event names, timestamps, a random per-launch ID,
and sanitized failure details so a crash or startup problem can be investigated
locally. The logger removes secret-bearing fields, URL destinations, absolute
paths, and control characters, and limits nested values and record sizes.
Renderer and child-process loss records contain only process type, reason,
exit code, numeric WebContents identity, and bounded Electron-supplied process
or service names when present. A runtime-environment record adds the Summer,
Electron, Chromium, and Node versions, operating-system family, architecture,
package kind, startup mode, and Electron's GPU feature status. It excludes GPU
hardware identifiers, driver paths, profile identifiers, URLs, and tab titles.
Summer does not enable raw crash dumps by default because they can contain
sensitive browser memory that cannot be reliably sanitized.

These ordinary diagnostics are not uploaded automatically. Three
files are retained with a maximum of 64 KiB each and older entries are replaced
automatically. The reporting guidance below still applies: do not share a raw
log unless a specific excerpt has been reviewed and sanitized.

The About settings include a local diagnostics viewer. It lets the user review,
filter, copy, clear, or deliberately export the sanitized rotating log. Summer
does not upload this general-purpose log automatically. Exported files should
still be reviewed before they are shared because even sanitized diagnostic
context may be sensitive.

## Optional technical crash reports

While crash reporting is enabled, each supported crash incident Summer detects is first written to a separate
durable queue and then sent automatically. A recovery choice—including restoring tabs, starting
fresh, Safe Mode, opening another browser, or quitting—does not suppress or
delay the report; the crash-reporting preference is changed in First Setup or About. An installed-package uploader runs as a separate detached
process, supervises the active browser run, and keeps retrying while Summer is
closed. Network failure leaves the report queued for bounded exponential retry.
A valid report remains queued until the collector accepts it or, if the queue
exceeds 128 reports, it is displaced as the oldest queued report. The local
queue has no separate age-based expiry.

The crash-report schema is deliberately closed. It contains an incident UUID
and timestamp; the exact release build ID and Summer, Electron, Chromium, and
Node versions; operating-system family, architecture, and package kind; process
class, an optional browser-chrome or web-tab role assigned by Summer's main
process, optional bounded Electron-supplied utility service name, bounded failure
reason and exit code; for GPU-process loss, a fixed list of broad feature modes
(enabled, software, disabled, unavailable, or unknown); startup mode; and path-free numeric
JavaScript frame locations when available. It does not contain URLs, tab titles,
browsing history, page or form contents, cookies, request headers, credentials,
tokens, clipboard data, prompts, profile identity, private-window state, adapter
names, vendor/device IDs, or driver versions.
Requests use HTTPS, omit browser credentials and referrers, and use the random
incident UUID only as an idempotency key. As with any direct HTTPS connection,
the collector and its network providers receive the source IP address and
ordinary connection metadata such as request time and transport characteristics.
The collector does not store the source IP address in the report envelope; it
uses only a temporary keyed value derived from the address in memory to enforce
rate limits.

To notify the operator without flooding the mailbox, the collector groups
reports by exact build ID, incident kind, process type, optional utility service
name, reason, exit code, optional fixed-catalog GPU feature modes, and a
server-owned UTC 24-hour bucket. It sends at most
one email through Resend to a Hike Tech Gmail mailbox for each matching cluster.
Expected child-process
`killed` and `memory-eviction` reports remain stored but do not send an email.
The notification contains those classification fields and the triggering
report's incident UUID, which lets the operator find that report without
exposing the full report in email. Later matching reports do not cause another
email, so their UUIDs are not included. The notification contains no stack
frames, source IP address, request headers, or request body. Resend and Google
receive that bounded classification and triggering UUID plus ordinary email
delivery and account metadata. Full reports remain accessible
only to authorized operators over SSH and are deleted from the collector after
30 days. The collector cannot delete the notification copies held by
Resend or the operator mailbox; their retention is controlled by those providers
and the operator's Gmail settings.

Because reports contain no account or profile identifier and Summer does not
currently show the incident UUID to the user, Hike Tech may be unable to locate
a specific report from a deletion request alone.

Hike Tech's operating direction is to use automated tooling for routine crash
classification, minimal local reproduction, fix verification, and timely
disposal rather than showing report payloads to developers. Automated output
must not expose sensitive report contents. This is an operating restriction,
not a claim that a developer can never access a report: authorized operators
may inspect the limited technical report when necessary for security, abuse,
collector reliability, or a crash that automation cannot diagnose.

Summer never uploads raw native crash dumps to its collector because process memory can contain
sensitive page or private-window data that cannot be reliably redacted. On
Windows, native-signature diagnostics is on by default for new and existing
profiles while ordinary crash reporting is enabled, unless the user turns it off.
Summer captures raw dumps only after verifying an owner-only Windows ACL on
their local directory; otherwise capture remains unavailable. It extracts bounded
native exception and module signatures, and queues each usable signature as a separate
technical report. A signature is not automatically linked to another crash
report when the relationship cannot be proven. The classified email may include
the exception code, fixed-catalog module name or category, and module-relative
offset. Dumps remain local and are never sent to the collector. Summer attempts
to delete dumps from a retired capture session on the next launch; deletion may
fail and is retried. Enabling or disabling native capture takes full effect only
after restarting Summer because the capture handler cannot be stopped during
the current session. Disabling ordinary crash reporting also disables the
separate native-signature choice. Turning off native-signature diagnostics
stops new native uploads, stops the background uploader, and attempts to remove
unsent native signatures while preserving ordinary reports. A signature already
accepted by the collector cannot be recalled. Windows Error Reporting is
governed separately by the user's Windows settings. The ordinary diagnostics viewer and its Clear action do not
delete the separate crash-report queue. Turning crash reporting off does delete unsent queued reports.

The uploader reads only the closed JSON queue. It never reads or uploads raw
crash dumps or the general diagnostics log. A profile-scoped lock prevents
duplicate uploaders, and incident IDs are stable across sentinel and next-launch
inference so collector retries remain idempotent. A simultaneous operating-
system or machine loss can still stop both processes; the durable queue and run
marker resume on the next launch.

When reporting is enabled, supported packaged desktop releases are configured to attempt delivery of these reports to
`https://crashes.summerbrowser.com/v1/reports`, operated by Hike Tech Ltd. Full
reports are limited to authorized Hike Tech operators over SSH and are deleted
automatically after 30 days, or earlier by an authorized operator. Resend and
the configured Gmail mailbox receive only the bounded classified notification
described above; retention of those notifications is controlled by those
providers and the operator mailbox.

This destination and its governance terms are checked into Summer's source and
validated by every distributable desktop release command. Each release embeds
them together with the exact source commit as its build ID and records the
non-secret values in release metadata. Packaged Summer uses only the embedded
values, so an installed user or another local process cannot redirect reports
through environment variables. Any future change to the destination, operator,
access controls, retention, or deletion policy requires an explicit source-code
and privacy-policy change reviewed before release. Development runs may use
test-only environment overrides. A source build without valid embedded
configuration keeps reports queued instead of sending them to an unknown
destination or uploading unattributed reports. Dirty local validation packages
embed neither the production endpoint nor a clean commit identity; their
reports remain queued locally and unattributed.

Summer also keeps a small versioned crash-recovery state in each profile. It
contains a consecutive-failure count, random run identifier, timestamps,
startup mode, and stability marker. It contains no URL, tab title, history, or
external-browser executable path. A stable run clears earlier failure counts
while retaining the active-run marker; a clean exit clears the record.

Separately, Summer keeps a bounded crash-session snapshot so it can offer to
restore regular tabs after a complete process failure. The snapshot contains
regular-tab URLs and titles, the active-tab identity, and Keep flags, with a
limit of 500 tabs and 4 MiB. Private windows are excluded. Page contents, form
values, cookies, renderer data, and Chromium navigation state are not stored in
this snapshot.

Summer's internal peer-to-peer work is not initialized during normal startup
and is not a public browsing feature.

## Website and bookmark artwork

The bundled Captain Word app can show official artwork for popular-domain
suggestions. When **Load website artwork** is enabled, Captain Word may contact
the public website and an artwork host declared by that website for each of the
at most five domain cards currently displayed. This can happen before the user
opens a suggestion.

First Setup can also request richer artwork for the bookmark suggestions that
are currently visible; it does not fetch the complete catalog eagerly. Added or
imported bookmarks without usable artwork can fetch a bounded portion of the
bookmarked page to discover its title and declared artwork. These setup and
bookmark-preview requests do not depend on Captain Word's **Load website
artwork** setting.

Captain Word and First Setup artwork requests contain no browser cookies,
referrer, browsing history, or raw address-bar query. The destination still
receives ordinary network information, including the user's IP address, the
requested URL, and Summer's dedicated website-artwork user agent. Summer
validates HTTPS destinations and public network addresses, bounds and rasterizes
responses, and gives the renderer only a local data URL. Bookmark-preview page
requests omit credentials and cache reuse, accept only HTTP or HTTPS pages, and
read at most 512 KiB of HTML, but the destination still receives ordinary
network information.

Successful artwork and page titles are cached in the active browser profile for
30 days; failed lookups are cached for six hours. Successful bookmark metadata
and artwork are stored with the bookmark, and a visible bookmark preview can be
refreshed after one week. Users can disable **Load website artwork** in Captain
Word settings to retain local address-bar domain cards without Captain Word's
requests; that setting does not disable First Setup or bookmark-preview requests.

## Language detection and page translation

Summer can offer translation when a page declares, or an already-installed
on-device detector identifies, a language outside the active profile's
understood languages. Page loading does not trigger a detector-model download.

Translation starts only after the user selects **Translate**. Summer may
download an allowlisted language model after that request and cache it locally.
The model host receives the ordinary network metadata required for that
download, but not the page text. Translation inference runs on the device.
Summer has no remote text-translation fallback, so page text is not uploaded to
a Summer or third-party translation service. Language and site exceptions
remain in the active local profile.

## What this policy does not cover

Summer is a browser. Websites and services that a user opens can receive the
same kinds of network information they would receive from another browser.
Search providers, operating-system Stores, extensions, and installed Summer
Apps are separate software and services with their own behavior and policies.

Summer Apps execute as trusted privileged code in Electron's main process.
They can access browser and operating-system capabilities and may communicate
with third-party services. Install only Apps you trust and review their source,
documentation, and data-handling disclosures.

## Contact

Questions and privacy reports can be sent to `talshalti1@gmail.com` or raised
through the reviewed Summer Discord link bundled in `summer://docs`.

Never include passwords, authentication tokens, payment-card details, private
profile data, or unredacted logs in an initial report.
