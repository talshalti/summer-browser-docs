# Summer App APIs

Summer Apps may expose bounded, versioned APIs to other Summer Apps and, when
explicitly declared, to ordinary websites through `window.summi`. AI Services
is the first intended provider, but the bridge is provider-neutral.

The bridge transfers validated data and calls registered main-process
functions. It never injects provider JavaScript into a consumer, exposes a
provider module, or grants raw Electron IPC.

## Manifest contract

An app may declare up to 16 APIs. API IDs are process-wide, stable identifiers;
resolution fails closed when more than one enabled app claims the same ID.
IDs beginning with `summer-` are reserved for exact bundled Summer Apps.

```json
{
  "apis": {
    "example-text-api": {
      "name": "Example text API",
      "description": "Transforms explicitly supplied text.",
      "version": "1.0",
      "audiences": ["summer-app", "website"],
      "methods": ["transform"]
    }
  }
}
```

An app consuming another app's API declares each exact ID:

```json
{
  "permissions": {
    "apis": ["example-text-api"]
  }
}
```

Declarations are not implementations. During activation, the provider must
register every declared method and no undeclared method:

```js
let disposeApi;

export function activate(context) {
  disposeApi = context.provideAPI("example-text-api", {
    async transform({args, signal, caller}) {
      const [text] = args;
      if (signal.aborted) throw new Error("Cancelled");
      return {text: String(text).toUpperCase(), caller: caller.type};
    }
  });
}

export function deactivate() {
  disposeApi?.();
  disposeApi = undefined;
}
```

Summer also unregisters implementations automatically when their activation
context is disposed.

## Summer App consumers

`context.loadAPI()` activates the unique enabled provider when necessary and
returns a frozen browser-created proxy. A missing, disabled, ambiguous,
undeclared, incompatible, or unauthorized API returns `null`.

```js
const textApi = await context.loadAPI("example-text-api");
const result = await textApi?.transform("Hello");
```

The requesting app must declare the API under `permissions.apis`. Loading an
API during a provider cycle fails closed; providers must not rely on circular
activation.

## Website consumers

Top-level HTTPS pages may request an API whose declaration includes `website`.
The call must begin during transient user activation. Summer identifies the
requesting hostname and providing Summer App before returning a proxy.

```js
button.addEventListener("click", async () => {
  const textApi = await window.summi?.loadAPI("example-text-api");
  const result = await textApi?.transform("Hello");
});
```

Refusal, absence, ambiguity, or incompatibility returns `null`. The handle is
bound to the requesting tab and origin, expires after five minutes, and is
invalidated by provider deactivation. It is never available to HTTP pages,
iframes, internal pages, popups outside the mapped tab model, or private tabs.

## Invocation contract

API methods are asynchronous. The provider receives one browser-created
invocation object:

```ts
interface SummerAppApiInvocation {
  caller:
    | {type: "summer-app"; id: string}
    | {type: "website"; origin: string};
  args: SummerAppApiValue[];
  signal: AbortSignal;
}
```

Arguments and results are cloned JSON-shaped values. Summer rejects cycles,
non-finite numbers, class instances, unsafe property names, excessive depth,
arguments above 64 KiB, and results above 256 KiB. Each declaration has at
most 32 methods. Calls have bounded concurrency and time, and are aborted when
the provider, consuming app, tab, or website connection disappears.

Provider exceptions become coarse bridge failures; provider error text does
not cross into websites. Providers must still validate their domain-specific
inputs, avoid secrets in results, honor cancellation, and make mutations safe
against uncertain completion.

## Compatibility and identity

The API ID identifies the contract; the provider app ID identifies who supplies
it. The API's `major.minor` version is reported on the proxy. Consumers should
request only APIs they understand and feature-detect optional methods. Breaking
changes require a new API ID or major version and an explicit compatibility
plan.

Summer does not enumerate APIs to websites or install a missing provider in
response to `loadAPI()`. Installing, enabling, disabling, updating, and
uninstalling remain explicit Summer App lifecycle actions.

## Browser-routed files

`manifest.fileHandlers` is the declarative entry point for user-dropped local
files. A handler declares an app-local `id`, safe lowercase final `extensions`,
and a `view` or `edit` intent. After review, the exact granted top-level app
page receives `window.summerFile`:

```ts
const file = await window.summerFile.open();
const response = await fetch(file.resourceUrl);
const bytes = new Uint8Array(await response.arrayBuffer());

if (file.intent === "edit") {
  await window.summerFile.save({bytes, revision: file.revision});
}
```

`open()` returns only `name`, final `extension`, `size`, declared `intent`, an
opaque read URL, and an opaque revision. It never returns a native path. `save()`
is available in the bridge for a uniform shape but fails closed unless the
manifest handler declared `edit`; editable files are limited to 4 MiB and use
external-change detection plus a synced same-file write that preserves the
file's existing filesystem identity and security metadata. A failed write uses
a best-effort synced restore of the original bytes.
