# Summer Browser 1.15.1

Released: July 20, 2026

Summer Browser 1.15.1 expands Summer Apps into configurable, widget-capable experiences; adds a complete Now Playing system; brings payment and multilingual profile autofill into the browser; and prepares Summer for distribution through the Mac App Store.

## Highlights

- Added a Now Playing widget that follows active audio and video across tabs, displays artwork or a live video preview, and provides playback, seek, volume, source-switching, and focus controls.
- Added app-provided widgets with sandboxed execution, host-mediated actions, responsive size presets, dragging, floating, docking, detaching, persistence, and automatic visibility controls.
- Added declarative settings for Summer Apps, including sections, conditional fields, validation, secrets, reset controls, and embedded configuration inside App Info.
- Added saved payment-card autofill with secure local storage, card management, field detection, and settings UI.
- Added Mac App Store packaging, entitlements, sandbox-aware runtime support, and automated release tooling.

## Summer Apps and widgets

- Apps can now bundle one or more widgets and expose them through the Apps & Widgets settings area.
- Added a reusable widget SDK for app context, host actions, dragging, idle appearance, and streamed canvas frames.
- Added widget permission boundaries and validation for app-provided HTML, scripts, styles, assets, and host commands.
- Added dedicated widget documentation, schema coverage, examples, and automated lifecycle, docking, sandbox, and media tests.
- Added configurable CaptainWord settings and migrated its options to the Summer App settings contract.
- Expanded App Info with a settings tab and made its descriptions, metadata, status badges, actions, image previews, and embedded settings fully readable in light mode.

## Autofill and browser import

- Added payment-card profiles, card-brand presentation, editing and deletion controls, and automatic credit-card form filling.
- Expanded name, address, contact, and payment-field recognition across more real-world form structures.
- Improved autofill for English, Hebrew, and Arabic forms, including localized field labels, address ordering, and profile presentation.
- Simplified saved-profile management and added clearer profile avatars, summaries, empty states, and responsive layouts.
- Added browser-data import to first-run onboarding and surfaced import actions when the browser has no bookmarks.

## macOS and Mac App Store

- Added Mac App Store targets, provisioning and entitlement support, universal packaging, validation, and release documentation.
- Added sandbox-safe downloads, sessions, file access, and security-scoped bookmark handling.
- Improved Safari bookmark migration and migration diagnostics.
- Added macOS-aware default-browser integration and a complete Chromium-compatible browser identity.
- Hardened find-in-page, tab restoration, widget behavior, and onboarding coverage for packaged macOS builds.

## Browser and media improvements

- Added the built-in `summer://error` page for navigation failures.
- Improved media-session tracking and browser media controls across tab activation and navigation changes.
- Preserved Store-created desktop shortcuts across application updates and improved shortcut launch handling.
- Added a dedicated Apps & Widgets settings experience for installed apps and active widgets.

## Fixes

- Fixed update detection that could incorrectly report an available update for the installed version.
- Fixed the saved-password autofill icon contrast.
- Fixed light-theme contrast throughout Summer App information and added a theme-aware foreground for primary accent actions.
- Fixed localized autofill profile ordering and presentation across left-to-right and right-to-left languages.
- Fixed desktop shortcuts so Store-managed installations continue to open the intended target after an update.

## Developer and release work

- Documented the Summer App settings and widget contracts and expanded the app manifest schema.
- Added focused coverage for cards, autofill profiles, widgets, app settings, CaptainWord, media controls, macOS integration, Safari migration, update policy, and release flows.
- Fixed the release UI type-check configuration so the complete quality gate can run from a clean checkout.
