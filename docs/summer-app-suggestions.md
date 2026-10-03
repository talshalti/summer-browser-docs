# Summer App suggestions

Summer Apps with the `SuggestionProvider` role export an asynchronous
`suggest(state)` function. Summer cancels the previous state when the address-bar
query changes and discards results from cancelled requests.
While the user continues typing within the same query, the desktop grid keeps
the last provider card mounted until the replacement arrives, so stable cards
update in place rather than briefly disappearing. A pending card is visual
only: it cannot be opened or used for Enter shortcuts or inline completion.
Starting an unrelated or empty query clears the previous presentation.

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

The built-in domain provider and phone website cards share local fallback artwork.
Known major brands use bundled vector marks on layered brand-color backgrounds;
uncatalogued sites receive deterministic geometric artwork instead of initials.
Captured site cards and validated provider images retain their existing priority.
Phone compositions omit the embedded desktop title, center the mark vertically
for wide search rows, and move it to the opposite side in RTL without mirroring
the logo. Artwork direction uses the resolved interface locale, including system
language tags such as `he-IL` and `ar-SA`, so it agrees with browser chrome.
This artwork adds no network requests when a query is typed.

The local marks come from the repository's existing MIT-licensed Bootstrap Icons
dependency. Regenerate their TypeScript catalog with
`node packages/captainword/scripts/update-domain-brand-marks.mjs`, then run
`npm --prefix packages/captainword run release` to update the desktop app assets.
Each generated SVG using those marks embeds the original license in metadata.

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
switches to the existing tab, while its bookmark actions, alias, tags, and
search metadata remain available. Protocol, subdomain, non-default port, path,
query, and fragment differences remain separate destinations. Multiple open
tabs remain separate switch targets even when they share a bookmark.

When an enabled installed app and an open tab have the same destination within
that app's own `sap://` authority, Summer shows the open-tab card once and keeps
the app's search terms, tags, and alias on it. Selecting it switches to the
existing tab. A different app, an external app action, or a different path,
query, or fragment remains separate; disabled-app discovery is unchanged.

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
LTR, Left Arrow for RTL); Enter keeps its existing address, search, alias,
and exclusive-provider behavior until a completion is accepted.

Completion reuses the already ordered local/provider result list and starts no
additional request. Search phrases, action-only results, widgets, credentialed
URLs, and unsupported schemes are not completion candidates. This keeps provider
actions explicit and prevents autocomplete from widening the navigation policy.
Installed-app cards are searchable by their `sap://` destination, so typing an
app-address prefix can reveal an on-request card and offer its inline completion.
Accepting a completion does not send its synthesized destination back to app or
extension providers; provider queries resume after the input changes again.
When an RTL interface begins receiving LTR text, Summer reserves the measured
suffix width on the right until input-direction detection finishes switching the
field to LTR, preventing mixed-direction text overlap during that transition.

Hovering a suggestion temporarily previews its address while preserving the
typed query and caret. Editing either the visible address field or its hidden
hover input ends that preview and resumes filtering with the committed edit.
Returning to the address field with the keyboard works while the pointer stays
over a card; leaving the card afterwards cannot restore the previous query.
IME composition keeps the existing results until the composed text is committed.

## Address field calculator

Desktop chrome shows an immediate calculator result below the address field for
complete arithmetic expressions, such as `2 + 3 * 4` or `(2 + 3) * 4`.
**Use result** inserts the answer and returns the caret to the end of the field
so another operation can be appended. Enter retains its usual search/navigation
behavior, and matching shortcuts retain priority.

The reusable `shared/addressCalculator.ts` parser accepts decimal and scientific
notation, parentheses, unary signs, `+`, `-`, `*`, `/`, right-associative `^`, and
postfix `%` (division by 100). Thus `200 * 10%` is `20`, while `200 + 10%` is
`200.1`. The symbols `×`, `÷`, and `−` and a trailing `=` are also accepted.
Powers bind before unary signs: `-2^2` is `-4`. Implicit multiplication and
functions are unsupported. Single numbers, incomplete/invalid expressions, and
non-finite results show no calculator row. Evaluation is synchronous and bounded
to 256 input characters and 128 tokens, with no JavaScript evaluation or added
network requests; the ordinary suggestion-provider flow is unchanged. Answers
use floating-point arithmetic; fractional answers are rounded to 15 significant
digits for display and reuse, while integer answers retain their represented
value. The result is LTR within both LTR and RTL interfaces.

## User display preferences

Summer's address view lets a user change a persistent installed-app or bookmark
card from **Always show** to **Appear by request**. An on-request card is absent
from the empty address view and appears when it matches entered text or a
selected tag. Its tags remain available while the card is hidden, and the user
can restore **Always show** from the card menu or the corresponding App or
Bookmark settings menu.

Bookmark Settings shows the profile's bookmarks in a searchable, paged card
grid using the same artwork presentation as address suggestions. Card
activation selects a bookmark for editing; it does not navigate. Selection can
span search pages, and the below-grid action adds one nonblank tag to each
selected bookmark that does not already have it (ignoring case). The existing
per-bookmark name and tag editor remains below the bulk action. A bookmark's
context menu still offers removal and address-card visibility, including by
keyboard; removing a bookmark clears it from the selection.

Built-in cards including **Extensions**, **File Explorer**, **Video Player**,
**CLI**, **Mesh**, and **Office Viewer** default to **Appear by request**.
Choosing **Always show** is a persistent
user override;
choosing **Appear by request** again removes that override and restores the
browser-owned default.

This is profile-local browser UI state. It does not modify an app manifest,
bookmark data, or provider output. A manifest's `showSuggestion: false` remains
an absolute app-author exclusion rather than a user display preference.

If a nonempty address-bar search has no current tab, bookmark, enabled-app,
widget, or provider results, the navigator may offer matching disabled installed
apps and uninstalled apps from the bundled registry. The fallback stays mounted
while the query changes, but its cards are disabled until current provider
results settle; any normal suggestion takes priority and hides it. Selecting
an offered app opens the Apps catalog filtered to that
app; installation and enablement still require the usual explicit user action
in app details. This does not make disabled apps executable from the address bar.

Widget cards default to **Appear by request**. A user can instead choose
**Don't show in address results** from a widget card's menu. That third state is
absolute for address discovery: the widget remains absent even when a query or
selected tag matches it. It does not close a running widget or uninstall its
provider. The complete Widgets catalog remains available in Settings, where
**Show in address results** restores the widget's request-driven behavior.

First Setup uses this same profile-local preference rather than changing app
installation or enablement. It offers enabled trusted apps only when they own a
static address-bar card; provider-only apps such as Captain Word are not choices.
A selected app or browser-widget card becomes **Always show**, while an
unselected card remains **Appear by request**. The widget card menu and Widgets
settings expose the same Always show, Appear by request, and hidden states.

In the desktop navigator, suggestion cards can also be deliberately dragged to
the removal tray at the bottom of the overlay. The card first follows with
elastic resistance, then breaks free and snaps into the tray only after a
downward drag reaches the target. Dropping a tab closes that tab, dropping a
bookmark removes the bookmark, and dropping an installed-app or widget card
changes it to **Appear by request** without disabling, uninstalling, hiding, or
closing it. Once the tray accepts the drag, the scaled card rests above the
tray with a clear gap so the action label remains unobstructed. An open-tab
result that also represents a bookmark closes only the tab and retains the
saved bookmark. Provider and store results do not activate
the tray. Cancelling the pointer gesture or releasing outside the target returns
the card to its original position. The magnetic transition lasts 280 ms and a
committed card finishes shrinking into the tray over 320 ms; reduced-motion
users receive the same result without those waits.

After a committed removal, the tray offers one **Undo** action for eight
seconds. Ctrl+Z activates it on Windows and Linux, and Command+Z does so on
macOS; normal text editing keeps those shortcuts when no removal undo is
available. A later removal replaces the earlier undo. Bookmark undo restores
the exact saved snapshot, while app and widget undo restores the exact prior
visibility. Tab undo becomes available only after the browser has recorded the
closed tab and restores that exact browser-owned record, including its
navigation history, strip position, and preserved state. Only authenticated
top-level browser chrome can resolve or restore that record. The existing card
menu remains the keyboard-accessible removal alternative. Closing the active
tab through this tray keeps the navigator open on the selected successor so its
Undo action remains available; ordinary active-tab closing still dismisses the
navigator. The tray marker remains pending while the native close reply is
awaited: a successor selection preserves the navigator only after the exact
marked tab has left the renderer's tab list. The navigator immediately advances
its tracked tab, so another ordinary selection dismisses it. A failed close or
unmounted suggestion grid clears its pending markers.
