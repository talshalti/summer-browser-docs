# Summer App suggestions

Summer Apps with the `SuggestionProvider` role export an asynchronous
`suggest(state)` function. Summer cancels the previous state when the address-bar
query changes and discards results from cancelled requests.

The role is descriptive in the current runtime: an app becomes a provider
only after it is active and Summer discovers its exported `suggest()`
function. There is no complete public always-on activation contract yet; the
bundled providers use the internal `lifeCycle: "perm"` field.

```ts
interface SuggestionQueryState {
  queryString: string;
  activeTags: string[];
  tagMatchMode: "any" | "all";
  bookmarkSuggestionCount: number;
  tabSuggestionCount: number;
  appSuggestionCount: number;
  abort: AbortController;
  invalidateSuggestions?: () => void;
}
```

`tagMatchMode` describes how a provider should combine `activeTags`: `"any"`
uses OR matching and `"all"` requires every active tag. Hosts that do not expose
tag filtering send an empty `activeTags` list with `"any"`.

## Progressive results

A provider should return useful local or cached results without waiting for an
optional remote lookup. It may start remote work with `abort.signal`, store the
result in its own bounded cache, and then call `invalidateSuggestions()` once.
Summer reruns the same request only if it is still current, and the provider
returns the newly cached data through its normal `suggest()` result.

The callback is optional for compatibility with older hosts. It is scoped to
one request, carries no query or suggestion data, becomes inert after
cancellation, and coalesces repeated calls. Providers must not call it while
serving unchanged cached data, because that would create a refresh loop.

```ts
export function suggest(state: SuggestionQueryState) {
  const cached = cache.get(state.queryString);
  if (!cached) {
    void loadRemote(state.queryString, state.abort.signal).then(results => {
      if (state.abort.signal.aborted) return;
      cache.set(state.queryString, results);
      state.invalidateSuggestions?.();
    });
  }
  return cached ?? immediateFallbacks(state.queryString);
}
```

Treat query text and remote payloads as untrusted. Bound cache size and lifetime,
validate remote URLs and result shapes, and avoid logging complete sensitive
queries.

## Exclusive local input claims

An exact canonical bundled app may return both `active: true` and
`exclusive: true` when a recognized protocol value must never fall through to
ordinary URL or search handling. Browser chrome prioritizes that result on
Enter. The main process strips `exclusive` from third-party suggestions, so a
manifest or downloaded provider cannot grant itself this authority.

Use this only for deterministic, locally validated protocol syntax. Return a
local error destination for malformed input with a recognized sensitive prefix
so that invalid identity material is not disclosed to DNS, navigation, or a
search engine.

## Opening a result

A returned `urlToOpen` is provider-controlled and does not currently pass
through a provider-specific protocol allowlist. Return only an intended
`https:` or `sap:` destination and validate any remote value before placing
it in a result. When a result has no URL, Summer calls the provider's
`openSuggestion({key, tabId, custom})` export instead.

A result may also expose context-menu actions. Prefer structured
`customActions` so the stable action ID remains separate from the user-visible
label. Summer passes the chosen `id` as `openSuggestion().custom`. The older
`customFunctions` string array remains supported, but uses each string as both
its label and action ID.

```ts
{
  key: "example",
  title: "Example",
  image: "data:image/svg+xml,...",
  imageVariants: {
    light: "data:image/svg+xml,...",
    dark: "data:image/svg+xml,..."
  },
  customActions: [{
    id: "hide-example",
    label: "Don't suggest this site"
  }]
}
```

`image` remains required as a fallback for older hosts. `imageVariants` may
provide a light override, a dark override, or both; Summer selects the matching
one at render time and falls back to `image` when it is absent. Keep each value
bounded like the base image and avoid embedding sensitive data.

Desktop and mobile run providers independently and in parallel through
`@summer/app-sdk/suggestions`. Results retain provider activation order, each
provider is bounded to 2.5 seconds, stale queries abort their provider
controllers, and malformed output, timeouts, or provider failures do not
discard healthy providers. Honor the abort signal and return promptly; the
timeout is a host safety bound, not a budget for routine remote work.

## Duplicate destinations

When an open tab and a saved bookmark resolve to the same complete normalized
URL, Summer shows one open-tab card with a bookmark badge. Opening the result
switches to the existing tab, while its bookmark actions, quick key, tags, and
search metadata remain available. Protocol, subdomain, non-default port, path,
query, and fragment differences remain separate destinations. Multiple open
tabs remain separate switch targets even when they share a bookmark.

Summer keeps an existing visible tab, bookmark, or installed-app suggestion when
an app-provided suggestion only repeats that HTTP(S) domain's homepage. Domain
comparison ignores a leading `www.` and the protocol, but keeps subdomains and
non-default ports distinct.

Only bare homepage suggestions are removed. Provider actions with a path, query,
or fragment remain available even when another suggestion uses the same domain.

## Inline address completion

Desktop Summer may show a muted inline suffix from the first currently visible
suggestion that has a prefix-matching browser address. The user accepts that
suffix with Tab or the forward arrow for the input direction (Right Arrow for
LTR, Left Arrow for RTL); Enter keeps its existing address, search, quick-key,
and exclusive-provider behavior until a completion is accepted.

Completion reuses the already ordered local/provider result list and starts no
additional request. Search phrases, action-only results, widgets, credentialed
URLs, and unsupported schemes are not completion candidates. This keeps provider
actions explicit and prevents autocomplete from widening the navigation policy.
Accepting a completion does not send its synthesized destination back to app or
extension providers; provider queries resume after the input changes again.
When an RTL interface begins receiving LTR text, Summer reserves the measured
suffix width on the right until input-direction detection finishes switching the
field to LTR, preventing mixed-direction text overlap during that transition.

## User display preferences

Summer's address view lets a user change a persistent installed-app or bookmark
card from **Always show** to **Appear by request**. An on-request card is absent
from the empty address view and appears when it matches entered text or a
selected tag. Its tags remain available while the card is hidden, and the user
can restore **Always show** from the card menu or the corresponding App or
Bookmark settings menu.

The built-in **Extensions**, **Video Player**, and **CLI** cards default to
**Appear by request**. Choosing **Always show** is a persistent user override;
choosing **Appear by request** again removes that override and restores the
browser-owned default.

This is profile-local browser UI state. It does not modify an app manifest,
bookmark data, or provider output. A manifest's `showSuggestion: false` remains
an absolute app-author exclusion rather than a user display preference.
