# Summer Browser privacy policy

Last reviewed for Summer 2.3.2.

Summer Browser does not operate telemetry, analytics, advertising, or
user-data collection services. Summer does not send browsing history,
credentials, autofill records, payment-card records, cookies, or private
profile data to a Summer-operated collection server.

## What this policy covers

This policy covers data collection performed by Summer Browser and services
operated by the Summer Developer Team. Browser profile data is kept on the
user's device unless the user deliberately exports or shares it.

## Local diagnostics

Summer keeps a small rotating diagnostic log inside each active browser
profile. It records bounded event names, timestamps, a random per-launch ID,
and sanitized failure details so a crash or startup problem can be investigated
locally. The logger removes secret-bearing fields, URL destinations, absolute
paths, and control characters, and limits nested values and record sizes.

These diagnostics are not uploaded or sent to a Summer-operated service. Three
files are retained with a maximum of 64 KiB each and older entries are replaced
automatically. The reporting guidance below still applies: do not share a raw
log unless a specific excerpt has been reviewed and sanitized.

The local diagnostics viewer is not exposed in Summer 2.4.0. Diagnostic files
remain internal to the current profile and must not be shared as raw logs.

Summer's internal peer-to-peer work is not initialized during normal startup
and is not a public browsing feature.

## Address-bar website artwork

The bundled Captain Word app can show official artwork for popular-domain
suggestions. When **Load website artwork** is enabled, Captain Word may contact
the public website and an artwork host declared by that website for each of the
at most five domain cards currently displayed. This can happen before the user
opens a suggestion.

These requests contain no browser cookies, referrer, browsing history, or raw
address-bar query. The destination still receives ordinary network information,
including the user's IP address, the requested URL, and Summer's dedicated
website-artwork user agent. Summer validates HTTPS destinations and public
network addresses, bounds and rasterizes responses, and gives the renderer only
a local data URL.

Successful artwork is cached in the active browser profile for 30 days; failed
lookups are cached for six hours. Users can disable **Load website artwork** in
Captain Word settings to retain local domain cards without these requests.

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
