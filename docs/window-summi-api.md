# `window.summi` website API

Summer exposes `window.summi` to the top-level document of ordinary HTTP and
HTTPS pages in normal mapped browser tabs. The API is not exposed to iframes,
`data:` pages, Summer internal pages, or other URL schemes. Standalone HTTP(S)
popups currently receive the session preload but are not mapped Summer tabs, so
their calls are rejected; treat popup use as unsupported.

The API is an implemented, per-request consent surface. It is not part of
Summer's persisted website-permission store and has no remembered Allow, Block,
or revoke state. Registration calls do not prompt. Open and cross-tab requests
normally prompt, with the same-tab shortcuts described below.

## Availability and profile scope

| Context | Result |
| --- | --- |
| Top-level HTTPS page in a normal Summer tab | API exposed and calls accepted |
| Top-level HTTP page in a normal Summer tab | API exposed for legacy methods; `loadAPI()` is rejected |
| Any iframe, including same-origin | API absent and main-process calls rejected |
| `summer:`, `sap:`, `file:`, `data:`, `about:`, or another scheme | API absent |
| Standalone top-level popup | Property may be injected, but calls are rejected; unsupported |
| Normal persistent profile | Registrations are kept in this browser process only |
| Separate `--instance-id` profile | Separate process-memory registry; no cross-instance discovery |
| Private or guest mode | No separate supported private/guest tab mode exists today |
| Another browser | API normally absent; keep a standard website fallback |

Feature-detect the global and every method used. Use HTTPS for real
integrations even though the current runtime also exposes the API on HTTP pages.

```js
const api = window.summi;
if (!api || typeof api.requestOpenWebsite !== "function") {
  // Keep the website's normal link or in-page behavior.
}
```

## Public contract

The authoritative declaration is [`types/summi.d.ts`](../types/summi.d.ts).
Summer does not yet publish a separate type package.

```ts
interface SummiWebsiteApi {
  readonly agentTools: {
    provide(definition: SummiAgentToolProviderDefinition): Promise<{
      readonly id: string;
      dispose(): Promise<void>;
    } | null>;
  };
  loadAPI(apiId: "extensions"): Promise<SummiExtensionsApi | null>;
  loadAPI(apiId: "captainword"): Promise<SummiCaptainWordApi | null>;
  loadAPI<ApiType extends object = object>(apiId: string): Promise<ApiType | null>;
  requestOpenEmail(email: string): Promise<boolean>;
  requestOpenService(serviceName: string): Promise<SummiActiveServiceHandle | null>;
  requestOpenWebsite(url: string): Promise<boolean>;
  registerActiveService(definition: {
    name: string;
    methods: Record<string, (...args: any[]) => any>;
  }): Promise<boolean>;
  registerEmailHandler(email: string): Promise<boolean>;
}

interface SummiExtensionsApi {
  readonly id: "extensions";
  readonly name: "Extensions";
  readonly version: "1.0.0";
  readonly provider: Readonly<{id: "summer-browser"; name: "Summer Browser"}>;
  readonly methods: readonly ["suggestExtension"];
  suggestExtension(extensionId: string): Promise<boolean>;
  call(method: "suggestExtension", extensionId: string): Promise<boolean>;
}

interface SummiCaptainWordApi {
  readonly id: "captainword";
  readonly name: "Captain Word";
  readonly version: "1.0.0";
  readonly provider: Readonly<{id: "summer-browser"; name: "Summer Browser"}>;
  readonly methods: readonly ["requestPersistence"];
  requestPersistence(request: {
    id: string;
    items: Array<{
      type: "direct" | "search";
      triggers: string[];
      image?: string;
      url: string;
    }>;
  }): Promise<"created" | "updated" | "unchanged" | null>;
}

interface SummiAgentToolProviderDefinition {
  id: string;
  tools: Array<{
    id: string;
    title: string;
    description: string;
    keywords?: string[];
    inputSchema: Record<string, unknown>;
    risk: "read" | "navigate" | "write" | "sensitive";
  }>;
  invoke(
    toolId: string,
    input: Record<string, unknown>,
    options?: Readonly<{signal: AbortSignal}>,
  ):
    Promise<{message: string; data?: unknown}> | {message: string; data?: unknown};
}

interface SummiActiveServiceHandle {
  readonly name: string;
  readonly methods: Readonly<Record<string, (...args: unknown[]) => Promise<unknown>>>;
  call(method: string, ...args: unknown[]): Promise<unknown>;
}
```

Normal policy refusals use coarse `false` or `null` results. IPC failures,
expired service connections, provider exceptions, timeouts, and serialization
failures may reject, so production callers still need `try`/`catch`.

## Provide tools to AI agents

Top-level HTTPS pages, plus loopback HTTP pages for local development, may
register tools that act only through the page's own JavaScript:

```js
const registration = await window.summi?.agentTools.provide({
  id: "mail",
  tools: [{
    id: "draft",
    title: "Draft an email",
    description: "Create a draft in the currently signed-in mail page.",
    keywords: ["email", "compose", "draft"],
    inputSchema: {
      type: "object",
      properties: {
        subject: {type: "string"},
        content: {type: "string"}
      },
      required: ["subject", "content"],
      additionalProperties: false
    },
    risk: "write"
  }],
  async invoke(toolId, input, options) {
    if (toolId !== "draft") throw new Error("Unknown tool");
    if (options?.signal.aborted) throw new DOMException("Cancelled", "AbortError");
    // Validate input again, then use this website's existing application code.
    return {message: `Created draft: ${String(input.subject)}`};
  }
});

// Optional; navigation or tab closure also invalidates the provider.
await registration?.dispose();
```

The browser assigns the public namespace from provider kind, owning tab, and
the declared provider ID. Registration does not grant browser authority. The
callback runs only when an agent selected a short-lived handle returned by the
browser-owned index, and it communicates through one private MessagePort call.
Summer rechecks the exact origin and live tab immediately before calling it,
limits each provider to 1,000 tools, limits the whole registry to 50,000 tools,
permits at most 16 combined active-service, email-handler, and agent-provider
registrations per tab, and permits at most four concurrent calls to one
webpage provider. Descriptors and callback results are JSON-normalized and
size-bounded in the isolated preload before IPC, then revalidated in main.

Agent prompts never contain the complete registry: search returns at most eight
ranked tools to AI Services, and the prompt builder drops descriptors that do
not fit the selected model's bounded context budget. Handles expire after one
minute, are scoped to the agent session and browser window, and are consumed
once. A webpage controls its callback and therefore cannot authorize itself by
declaring a low risk: every webpage-provided descriptor is treated as
`sensitive` at the trusted boundary. The user's AI Services agent policy decides
whether sensitive actions run without another prompt, ask first, or are blocked.
When Summer asks, the native dialog reports the browser-owned exact website
origin and location of the tab. When Summer cancels an invocation, the
callback's optional `options.signal` aborts so cooperative provider work can
stop. Provider calls time out after ten seconds; a timeout still only stops
waiting and cannot cancel JavaScript that has already started in the provider
document, so the outcome is reported as unknown and is not automatically
retried.

## Consent behavior

- `registerActiveService`, `registerEmailHandler`, and agent-tool provider
  registration do not prompt. Registration itself grants no browser action.
- Invoking a webpage tool follows the profile's **AI Services → Agent approval
  settings → Sensitive and provided tools** decision. The dialog, when shown,
  uses the origin and tab identity supplied by browser chrome, not page metadata.
- `loadAPI` requires transient user activation. Installed Summer App APIs prompt
  before returning a short-lived package-backed proxy. The browser-provided
  `extensions` and `captainword` APIs skip that preliminary prompt because their
  methods show the browser-owned review flow and require fresh activation too.
- `requestOpenWebsite` prompts for every valid request.
- `requestOpenEmail` normally prompts before switching or opening. It returns
  `true` without a prompt if the requesting tab is already the exact live custom
  handler or matches the inferred provider hostname.
- `requestOpenService` normally prompts before switching and issuing a handle.
  A tab requesting its own registered service receives a handle without a
  prompt.
- One prompt may be pending per requesting WebContents/tab. Two tabs from the
  same origin can each prompt, while a second request in one tab is refused.
- A notification closes after five idle seconds. Pointer, keyboard, wheel, or
  click activity refreshes that timer.
- Approval for a service handle covers every method advertised by that handle;
  calls do not prompt again during the handle's lifetime.

Website prompts show requester and target hostnames. Service prompts show the
requester and registered-service hostnames. Email prompts show the full
requested address plus a provider label or custom-handler hostname. No prompt
shows a complete destination URL, authenticated page or account identity, or
method-level grant. Hostnames omit scheme, port, path, and query, so HTTP and
HTTPS pages or alternate ports on the same hostname look identical.
A visible prompt is not currently cancelled or rebound when its requester
navigates or closes, so prompt text is not proof that the original page remains
live when the user acts.

## Email routing

### Request an inbox

```js
export async function openInbox(email) {
  const api = window.summi;
  if (!api) return {supported: false, opened: false};

  try {
    return {supported: true, opened: await api.requestOpenEmail(email)};
  } catch {
    return {supported: true, opened: false};
  }
}
```

Summer maps known domains to their webmail hosts, including Gmail, Outlook,
Yahoo Mail, iCloud Mail, Proton Mail, Fastmail, Zoho Mail, AOL Mail, Yandex
Mail, GMX Mail, and Mail.com. Unknown domains fall back to
`https://mail.<domain>/`.

An existing provider tab is matched by hostname, not by signed-in account, and
only inside the same privacy session. Summer can focus the provider tab's owning
window rather than creating a duplicate in the requester's window. A `true`
result means Summer was already at a matching tab or completed an approved
switch/open action; it does not prove that the requested account is signed in or
that the destination finished loading.

Browser-owned `mailto:` handling reuses saved autofill email only in normal
windows. Summer asks which profile to use when multiple distinct addresses
exist, and closing that chooser cancels the action. Gmail and Outlook currently
have reviewed compose handlers that receive the complete normalized `mailto:`
URI. Other known providers, unknown domains, invalid or missing profile data,
and private windows use the operating-system mail handler. Browser-owned
`mailto:` handling never trusts a custom Summi email registration.

On Windows, a packaged Summer installation also registers as an available
`mailto:` handler. The user must still select it in Windows Default Apps. An
externally activated mail link uses the same saved Gmail or Outlook compose
route. If that route is unavailable, Summer shows a bounded notice and never
delegates the URI back to itself.

### Register a custom inbox

```ts
export async function registerThisInbox(email: string): Promise<boolean> {
  const api = window.summi;
  if (!api) return false;

  try {
    return await api.registerEmailHandler(email);
  } catch {
    return false;
  }
}

void registerThisInbox("support@company.example");
```

Custom handlers are available only for unknown providers. They match one exact
address case-insensitively, not an entire domain. A later eligible tab can
silently replace the same address registration. Register after every document
load and keep a normal email link as the fallback.

## Website requests

```js
export async function openHelpWebsite() {
  const api = window.summi;
  if (!api) return {supported: false, opened: false};

  try {
    const opened = await api.requestOpenWebsite("https://example.com/help");
    return {supported: true, opened};
  } catch {
    return {supported: true, opened: false};
  }
}
```

The destination must be a 1â€“4096 character absolute HTTP or HTTPS URL without
an embedded username or password. Plain hostnames, script URLs, and Summer
internal schemes are rejected. A `true` result means the approved tab-opening
action ran, not that the page loaded successfully.

## Browser-provided Extensions API

An extension publisher can ask Summer to show the same browser-owned install
recommendation used on Chrome Web Store pages and curated official sites:

```js
installButton.addEventListener("click", async () => {
  const extensions = await window.summi?.loadAPI("extensions");
  const shown = await extensions?.suggestExtension(
    "jldhpllghnbhlbpcmnajkpdmadaolakh"
  );
  if (shown === false) {
    // Keep the normal Chrome Web Store link available.
  }
});
```

Both `loadAPI("extensions")` and `suggestExtension()` require current transient
user activation in a top-level HTTPS page. The ID must be one exact lowercase
32-letter Chrome extension ID (`a` through `p`). `true` means the recommendation
visibly opened; `false` covers invalid input, stale/private/background callers,
an unavailable manager, an installed extension, rate or snooze policy, another
prompt, and notification failure. It does not mean the user chose to install.

The website supplies only the extension ID. Notification names come from
Summer's browser-owned catalog when known; arbitrary website text, icons, URLs,
and install packages are never accepted. The immutable proxy carries no install
authority, and every call is revalidated in the main process. **Review &
install** still mints the short-lived single-use manager token, downloads the
signed Web Store package through the existing path, opens the normal requested-
access review, and requires a second explicit confirmation. **Not now**, the
30-day snooze, the profile opt-out, proactive-tip budgets, private-window
exclusion, and already-installed suppression are shared with automatic
recommendations.

## Browser-provided Captain Word API

An HTTPS website can offer static address-bar routes that remain useful after
the website's tab is closed. Call the API from a visible user action:

```js
addToSummerButton.addEventListener("click", async () => {
  const captainWord = await window.summi?.loadAPI("captainword");
  const result = await captainWord?.requestPersistence({
    id: "youtube",
    items: [
      {
        type: "direct",
        triggers: ["youtube", "yt"],
        url: "youtube.com"
      },
      {
        type: "search",
        triggers: ["youtube", "yt"],
        url: "https://www.youtube.com/results?search_query=%%"
      },
      {
        type: "direct",
        triggers: ["youtube shorts", "yt shorts"],
        url: "https://www.youtube.com/shorts/"
      }
    ]
  });
  if (!result) {
    // Keep the website's ordinary navigation and search controls available.
  }
});
```

The spellings are `loadAPI` and `requestPersistence`. `loadAPI("captainword")`
returns a frozen browser-created proxy without prompting. The persistence call
requires fresh transient activation and opens a browser-owned review containing
the browser-derived requester hostname plus every trigger and destination. A
changed registration is reviewed again; an identical registration returns
`"unchanged"` without another prompt. `"created"` or `"updated"` means the
reviewed snapshot was saved. `null` covers invalid data, refusal, stale or
private callers, an unavailable canonical Captain Word package, quota or
storage failure, and a changed page while the review was open.

Registrations are owned by the exact HTTPS origin plus the normalized `id`.
Another origin cannot replace them. Each request accepts 1–16 items and each
item accepts 1–8 bounded triggers. A direct URL may be an HTTPS URL or a bare
host that Summer upgrades to HTTPS. A search URL must be HTTPS and contain
exactly one `%%`, which receives the percent-encoded text after the trigger.
Destinations must match the requesting hostname; only an optional leading
`www.` is treated as equivalent. Credentials, other schemes, unknown fields,
duplicate type/trigger pairs, and unrelated destinations are rejected.

Omit `image` to let Captain Word resolve and cache the destination's normal
artwork. To prevent a persisted entry from becoming a tracking request after
the source site closes, website-supplied images are limited to bounded base64
PNG, JPEG, or WebP data URLs; live remote image URLs are rejected.

Exact direct triggers win before searches, and the longest trigger wins. Thus
`youtube shorts` opens Shorts rather than searching YouTube for `shorts`.
Partial direct matches can display completion cards but never take over Enter.
The saved item is ordinary static data: Captain Word never calls the source
website, receives its cookies, or runs website JavaScript. Users can review the
source, triggers, and destinations in Captain Word's panel in Summer Settings,
then remove the website extension there. Removal revokes every registered ID
owned by that exact origin. Right-clicking one of its suggestions and choosing
**Remove** provides the same origin-wide revocation path.

## Summer App APIs

For API IDs other than the browser-provided `extensions` and `captainword` APIs,
`loadAPI(apiId)`
resolves one exact API declared by an enabled installed Summer App. It returns
`null` when the ID is invalid, missing, disabled, ambiguous, incompatible, not
website-enabled, refused, or not registered during provider activation. Summer
never enumerates APIs or installs a missing provider.

```js
button.addEventListener("click", async () => {
  const ai = await window.summi?.loadAPI("summer-ai-adapter");
  const result = await ai?.generate("Explain this in one paragraph.");
  if (result) output.textContent = result.text;
});
```

Only top-level HTTPS pages in normal mapped tabs may load an API. The preload
requires current transient user activation. Consent identifies the requesting
hostname, API, and providing Summer App. The provider declaration must include
the `website` audience.

The returned object is a frozen browser-created RPC proxy containing direct
method aliases, `call(method, ...args)`, the API ID and version, and provider
identity. It contains no provider JavaScript, module reference, Electron object,
IPC channel, credentials, or browser authority. Handles expire after five
minutes and are bound to the requester tab and origin. Calls use bounded cloned
JSON values and fail after navigation, provider replacement, disablement,
uninstall, timeout, overload, or cancellation.

The complete package declaration, provider, app-consumer, validation, and
lifecycle contract is documented in
[`summer-app-apis.md`](summer-app-apis.md).

## Active services

### Register a provider

```js
function validCustomerId(value) {
  return typeof value === "string" && /^[A-Za-z0-9_-]{1,64}$/.test(value);
}

export async function registerSupportService() {
  const api = window.summi;
  if (!api) return false;

  return api.registerActiveService({
    name: "Example Support",
    methods: {
      openConversation(customerId) {
        if (!validCustomerId(customerId)) {
          throw new TypeError("Invalid customer ID");
        }
        return {opened: true, customerId};
      }
    }
  });
}
```

A service name is NFKC-normalized, trimmed, whitespace-collapsed, and looked up
case-insensitively. It contains 1â€“64 characters, starts with a Unicode letter or
number, and otherwise permits letters, numbers, spaces, dots, underscores, and
hyphens. It is a process-wide, last-writer-wins labelâ€”not an owned, verified, or
authenticated namespace. A later eligible tab can replace it without
registration consent.

Each definition must contain 1â€“32 unique function-valued methods. Method names
are case-sensitive and must match `[A-Za-z_$][A-Za-z0-9_$]{0,63}`;
`__proto__`, `constructor`, and `prototype` are rejected. A tab may hold at
most 16 combined service and custom-email registrations.

### Request and call a service

```ts
export async function openSupportConversation(customerId: string): Promise<unknown | null> {
  const api = window.summi;
  if (!api) return null;

  try {
    const service = await api.requestOpenService("Example Support");
    if (!service) return null;
    return await service.call("openConversation", customerId);
  } catch {
    return null;
  }
}
```

`handle.call(method, ...args)` is the stable dynamic call surface.
`handle.methods.<name>(...args)` is a browser-generated convenience alias for a
method advertised at approval time. The alias name belongs to the provider's
contract; it is not an additional stable Summer API method.

A handle:

- is bound to one requesting tab and one exact registration;
- expires after an absolute five minutes; calls do not renew it;
- is invalidated by either tab closing, service replacement/re-registration,
  or a newer handle from the same requester to the same service;
- lets Summer await at most eight responses at once;
- stops awaiting each response after ten seconds; and
- accepts only structured-cloneable arguments and results.

Those limits do not cancel provider JavaScript. A timed-out callback can keep
running while newer calls are accepted, so eight is not a hard provider
concurrency or call-rate limit. The API also defines no smaller payload-size
limit beyond what structured cloning and the process can handle. Providers must
enforce their own payload-size and rate limits and make state-changing methods
idempotent or deduplicate request IDs. Callers must not blindly retry a timed-out
mutation because the first callback may still complete.

Active-service callbacks are document-local, but main registrations are
origin-and-tab records. Register after every page load. A same-origin reload can
leave a stale descriptor discoverable, and a failed callback does not delete
it. Leaving and returning to the origin before a liveness lookup can also keep a
record. Closing the provider tab is the only direct way to make its name
unavailable; navigation alone is not reliable. Replacement invalidates old
handles but keeps the service name available through the replacement. None of
timeout, expiry, requester-tab close, or replacement cancels a provider callback
already running; those conditions reject or disable calls only when observed,
and a call begun before expiry or replacement can still finish. Closing the
provider tab destroys its execution context. There is no explicit `unregister`,
`disconnect`, or manual permission-revocation method.

## Security and privacy

- Consent is not authentication. A registered website callback receives only
  caller-controlled positional argumentsâ€”no caller origin, page identity,
  service-routing metadata, or user identity.
- One approval covers the handle's complete advertised method set. Expose
  narrow methods and require an in-service confirmation for sensitive or
  account-changing actions.
- Validate every argument again in the provider and return only bounded
  structured data.
- Do not send passwords, tokens, cookies, private messages, or other secrets
  through service calls. A provider's thrown error message is returned to the
  caller after truncation, so error text must not contain sensitive data.
- Service names and exact custom-email addresses can be silently replaced and
  must not be treated as identity or ownership.
- Never assume iframe availability or a user gesture. Initiate prompts from a
  visible user action and keep least-privilege fallback behavior.

## Failure results

| Result | Meaning and response |
| --- | --- |
| `window.summi` is `undefined` | Ineligible document, another browser, or an older Summer build; use the standard fallback |
| Boolean method returns `false` | Invalid/unsupported input, registration limit, known-provider rejection, dismissed or concurrent prompt, suppressed extension recommendation, notification failure, or failed action; do not retry in a loop |
| `requestOpenService` returns `null` | No live service or no approved connection; keep the page usable without it |
| Promise rejects | IPC failure, expired/invalid connection, replacement, provider error, overload, timeout, or serialization failure; catch and use a bounded fallback |
| Old handle begins rejecting | Five-minute expiry, tab close/navigation, newer handle, re-registration, or provider failure; request a fresh handle only from a new user action |

## Compatibility

`window.summi` was introduced in Summer 1.15.0 and remains experimental. It has no API
version field, version negotiation, external type package, formal deprecation
window, background/service-worker API, iframe API, caller-origin argument,
owned service namespace, or explicit unregister API.

Feature-detect the global and every method, treat `false` and `null` as normal
refusals, catch rejected promises, never persist handles, and test against the
exact Summer version the website supports.
