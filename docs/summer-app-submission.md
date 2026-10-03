# Summer App submission and review

Summer Apps are public, supported features. They run inside Summer's
privileged Electron main process, so installing one grants it the same practical
access as trusted desktop code. The Summer Developer Team reviews packages
before listing them in the bundled Store registry.

## Official intake

Send a review request through one of these owner-approved channels:

- Discord: <https://discord.gg/MjCjcqPhX3>
- Email: `talshalti1@gmail.com`

Do not send passwords, tokens, private user data, signing keys, or unredacted
logs. There is currently no review-time guarantee or formal appeal workflow.

## What to submit

Submit the package as the same ASAR artifact users will receive. Also provide a
public source repository containing:

- the source code for the submitted version;
- complete build and packaging instructions;
- documentation for the app's behavior, supported operating systems, and
  known limitations.

Use Electron's ASAR tooling to create the package:

```powershell
npx @electron/asar pack app-directory app.asar
```

Summer does not currently add a signature, artifact hash, or other package
transformation. The submitted artifact is reviewed and received as-is.

### Developer-only GitHub installation

Desktop builds with Developer tools enabled can prepare an app from a public
GitHub repository, latest-release URL, or exact release-tag URL. Summer never
clones or builds repository source. It asks GitHub for the selected published
release, requires exactly one uploaded `.asar` asset, caps the download at
100 MiB, checks its declared size, and verifies GitHub's SHA-256 asset digest
when GitHub supplies one. It then validates the root `sap.json`, current desktop
platform, and declared main entry before showing the install confirmation.

This is a developer convenience, not a review or trust guarantee. The GitHub
repository, TLS download, and asset digest do not prove that the package matches
its public source or that its privileged code is safe. The main process enforces
the Developer tools gate independently of the renderer, and a confirmed package
is installed disabled. Enable it separately only after reviewing the source and
release. GitHub release provenance is retained with the installed record; Summer
does not currently poll GitHub or update the app automatically.

## Current app contract

The current installer requires `name`, `id`, `main`, `version`, and a non-empty
`type` array. The intended future direction is that only `name` should be
required, but changing identity fields safely requires an explicit migration
because app IDs currently anchor routes, settings, secrets, widgets, and
replacement behavior.

`name` is also the required compatibility fallback for the app's display name.
An app may provide translated names with `localizations`, keyed by BCP 47
language tags:

```json
{
  "name": "Review Example",
  "localizations": {
    "he": {"name": "דוגמת סקירה"},
    "ar": {"name": "مثال للمراجعة"}
  }
}
```

Summer resolves the exact active locale, then its base language, then `name`.
Older hosts that do not use `localizations` continue to show `name`, so it must
remain meaningful and must not be removed. Localized names affect display only:
do not translate or change `id`, which remains the stable authority for routes,
settings, permissions, stored data, updates, and uninstall. Reviewers should
check each submitted translation and may ask for unreviewed or misleading
entries to be removed.

Manifest roles describe how Summer should host and present an app, and
communicate that role to the user. They are not permission boundaries. Most
legacy manifest permissions remain disclosures because installed app modules
run in Summer's privileged main process. The API bridge is an exception:
`permissions.apis` is enforced before `context.loadAPI()` resolves a provider.

`version` is the user-visible app release. Each declaration under `apis` has an
independent `major.minor` contract version returned to consumers. The legacy
top-level `apiVersion` field remains metadata and is not negotiated. Breaking
API changes require a new ID or major version and an explicit compatibility
plan; Summer's general target remains roughly three months before a deprecated
public contract is removed.

Every new app must set all six `platforms` booleans in `sap.json`: `windows`,
`ios`, `macos`, `linux`, `android`, and `any`. A missing declaration supports
Windows, macOS, and Linux only for compatibility with older packages. A host
requires its matching value to be exactly `true`, unless `any` is `true`; `any`
enables every named host.
The declaration is a claim, not proof of compatibility, so the documentation
and review request must name the operating systems and host versions actually
tested. Navigation inside an app is the app's responsibility.

### Minimal reviewed source shape

```text
review-example/
|-- sap.json
`-- main.mjs
```

```json
{
  "name": "Review Example",
  "localizations": {
    "he": {"name": "דוגמת סקירה"},
    "ar": {"name": "مثال للمراجعة"}
  },
  "id": "review-example",
  "main": "./main.mjs",
  "version": "0.1.0",
  "description": "A minimal offline page for source-level testing.",
  "platforms": {
    "windows": true,
    "ios": false,
    "macos": true,
    "linux": true,
    "android": false,
    "any": false
  },
  "type": ["SummerApp"],
  "tags": ["summer:apps"],
  "keywords": ["example", "offline"]
}
```

```js
export function activate() {}

export function deactivate() {}

export async function serve(request) {
  const url = new URL(request.url);
  if (url.pathname !== "/") return new Response("Not found", {status: 404});
  return new Response(
    "<!doctype html><meta charset=\"utf-8\"><title>Review Example</title><h1>Review Example</h1>",
    {headers: {"content-type": "text/html; charset=utf-8"}},
  );
}
```

## Eligibility and review

Packages are eligible unless they are harmful or unlawful. The review is
performed by the Summer Developer Team and is proportional to the app's
complexity and the developer's experience. Review evidence normally includes
the source code, reproducible build instructions, documentation, and reasonable
behavior checks.

The review asks:

1. Does the submitted ASAR correspond to the provided source and build steps?
2. Does the app do what its documentation says?
3. Does it comply with applicable law?
4. Does it avoid harmful, deceptive, or abusive behavior?
5. Are its platform limits and privileged capabilities disclosed clearly?

`verified` means the Summer Developer Team reviewed the app. `featured` means
the team genuinely recommends it for the particular Summer user who will see
the recommendation. Neither flag creates a sandbox or guarantees that an app is
free of defects.

## Registry and lifecycle

`public/appregistry.json` is the Store discovery catalog bundled with Summer.
Its top-level version is bookkeeping for maintainers. Registry entries are
verified by the Summer Developer Team before publication.

- `updateUrl` is metadata; Summer does not yet provide an automatic third-party
  app updater.
- Installing another package with the same ID replaces the installed record
  without enforcing a higher version. Settings, protected secrets, and the
  previous enabled state are preserved.
- Removing an entry from a future Store catalog does not remotely uninstall an
  already installed copy.
- User uninstall removes the local app record, settings, and loaded app.

If an app becomes harmful or seriously broken, the immediate response is to
investigate and fix the underlying problem, publish a corrective update when
possible, and remove it from the Store when appropriate. Summer does not
currently have automatic remote revocation or denylist enforcement.

## Platform notes

- Windows is distributed through the Microsoft Store.
- macOS support is currently in the workshop stage and targets the Mac App
  Store. The target is feature parity except for Widevine at first, subject to
  change as the Store work develops.
- Linux support is planned.
- No public portable builds are currently offered.

These statements describe direction, not a guarantee that an app has been
tested on every platform.

## Review preparation worksheet

```markdown
# Summer App review preparation

- Source repository, commit, and local build steps:
- App ID, version, roles, and exact Summer build tested:
- Runtime files and privileged capabilities actually used:
- Network destinations and data handled, or none:
- License and third-party assets/dependencies:
- Sanitized screenshots and known limitations:
- Install, activation, routing, failure, cleanup, and uninstall results:
- Reviewer questions and follow-up:
```

Questions about the process can be sent through the same Discord or email
channels listed above.
