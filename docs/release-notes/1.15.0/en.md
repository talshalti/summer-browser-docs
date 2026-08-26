# Summer Browser 1.15.0

Released: July 15, 2026

Summer Browser 1.15.0 introduces a complete light theme, faster bookmark management, richer app details, PDF support, and safer website permissions.

## Highlights

- Choose between dark, light, and system-following themes throughout the browser, welcome page, settings, native title bar, and translucent navigation overlay.
- Review Chrome or Safari bookmark folders before importing them, then browse large bookmark collections with faster pagination and lazy-loaded artwork.
- Open PDF links directly in Summer and get faster, more reliable tab previews.
- View richer Summer App information, including descriptions, permissions, links, images, and available actions.
- Control WebAuthn requests before Windows Security opens, with remembered per-site permission choices.

## More improvements

- Added a tab-loading indicator and smoother card and navigation animations.
- Added bookmark search and editing improvements for large collections.
- Added `window.summi`, a permission-aware website API for cross-tab services and email routing.
- Improved Hebrew translations and polished onboarding, app information, icons, and desktop shortcuts.
- Expanded automated release validation, including ESLint, browser smoke tests, profile-upgrade coverage, and focused unit tests.

## Fixes

- Restored reliable loading of the built-in welcome page and improved its light-theme contrast.
- Fixed imported bookmarks whose URLs did not include an explicit scheme.
- Preserved YouTube favicons across reloads and navigation failures.
- Kept typing responsive while hovering navigation suggestions and cleared stale queries when hovered tabs close.
- Prevented saved credentials from being removed merely because duplicate entries exist.
- Fixed a settings divider that could disappear and improved notification sizing.
- Mirrored native window controls and their safe area for Hebrew and Arabic layouts.
- Corrected Windows update versioning for releases after 1.14.0.
