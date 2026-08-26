# Profile languages and page translation

Summer keeps one ordered language preference list per browser profile. The
list belongs to the profile, not to the browser installation or an individual
Summer App. It is configured under **Settings > Appearance > Languages**.

A new profile starts with its active Summer interface language as its only
understood language and translation target. Adding, removing, or reordering
languages does not change the interface language. The first remaining language
becomes the translation target if the current target is removed.

The stored `appearance.languagePreferences` value has this shape:

```ts
interface ProfileLanguagePreferences {
  understoodLanguages: string[];
  translationTarget: string;
  offerTranslation: boolean;
  neverTranslateLanguages: string[];
  neverTranslateSites: string[];
}
```

Language values are canonical BCP 47 tags. A general language preference such
as `en` accepts regional variants such as `en-GB`; an explicit writing system
such as `zh-Hant` remains distinct. The main process validates and bounds every
array before storing or exposing it. Site exceptions are normalized hostnames
and also cover their subdomains.

## Summer App integration

Every active Summer App receives a read-only live view:

```ts
const current = context.languagePreferences.get();
const stop = context.languagePreferences.onDidChange(preferences => {
  // Reconfigure language-sensitive behavior.
});
```

The API deliberately exposes only the ordered understood languages,
translation target, and translation-offer preference—not site or language
exceptions, renderer settings, or raw IPC. Apps decide which of their supported
language packs correspond to the ordered profile list. An app may offer a
clearly labeled app-specific override when its language needs differ from
general browsing.

Captain Word uses the profile list by default. Its **Suggestion languages**
setting can switch to a custom list without modifying the browser profile.
Legacy per-language values are ignored unless custom mode is explicitly
selected. Captain Word also has an independent **Suggest regional websites**
switch. Its global popular-site coverage defaults to the Top 100 tier;
explicitly configured wider tiers remain unchanged.

## Translation offer flow

After an HTTP(S) main-frame load, Summer checks the page's declared language.
When no language is declared, Summer may use Chromium's on-device language
detector only if its model is already available. Merely loading a page never
starts a detector-model download.

Summer shows a browser-owned translation bar only when all of these conditions
hold:

- translation offers are enabled;
- the detected language is not understood;
- neither the language nor the site is in the profile exceptions;
- the page has a valid HTTP(S) URL.

The user can dismiss the offer for that navigation, permanently suppress offers
for the language or site, or choose any understood language as the target.
The translation options also let the user add the page language directly to
**Languages you understand**. That profile preference takes effect immediately
and suppresses future offers for the language. Exceptions and understood
languages remain editable in Appearance.

Translation begins only after the user selects **Translate**. Summer resolves
the request to a revision-pinned model with an explicitly tested execution
policy. Korean-to-English uses an approximately 125 MB OPUS-MT model and
batches up to eight similarly sized text nodes. Russian-to-English retains its
approximately 120 MB OPUS-MT fast path and translates sequentially because that
model's batched decoder can introduce padding artifacts.

Other covered directions use M2M100, an approximately 650 MB model for direct
many-to-many translation between 100 languages, including Arabic, Chinese,
English, French, German, Hebrew, Hindi, Japanese, Korean, Portuguese, Russian,
and Spanish. It supports 9,900 directions without routing non-English pairs
through English. Its current ONNX decoder is deliberately kept sequential:
batching was measured as slower and corrupted output in this runtime.

Model and tokenizer data downloads from Hugging Face into a cache shared by the
installation's browser profiles. Later requests using the same model reuse the
cache. Inference runs in a worker, so page text remains on the device and does
not block the browser's main event loop. The UI reports model-download and
translation progress. After a short measurement window, it also shows a live
estimated-time-remaining countdown based on the current phase's observed
progress rate. Results are restored to their original DOM-node order before
they are applied.

Latin-script product names, URLs, and identifiers embedded in Russian prose are
preserved verbatim and the surrounding Cyrillic segments are translated
independently.

Pairs outside the multilingual model's coverage may use Chromium's
feature-detected on-device Translator API. Summer aborts and stops waiting if
Chromium cannot prepare its model within ten seconds. There is no remote
text-translation fallback. If neither Summer nor Chromium has a provider,
Summer reports that state instead of sending page text elsewhere. Summer-owned
models must be explicitly allowlisted and revision-pinned in
`shared/onDeviceTranslationModels.ts`.

ONNX Runtime 1.24 no longer publishes a macOS Intel binary. On Intel Macs,
Summer therefore routes all translation requests through Chromium's
feature-detected on-device Translator API; Apple-silicon Macs continue to use
the Summer-owned model paths above.

To keep work bounded and reversible, one request translates at most 300
text nodes and 60,000 characters. Summer skips scripts, styles, form controls,
code, preformatted content, SVG, MathML, and editable content. **Show original**
restores the text nodes that still belong to the current document. Dynamic text
added later is not automatically translated in this first implementation.

## Process boundaries

- `shared/languagePreferences.ts` owns validation, matching, defaults, and offer
  policy.
- `electron/main/settings.ts` persists validated profile data and updates the
  live main-process service.
- `electron/main/apps/apploader.ts` exposes the read-only Summer App capability.
- `electron/main/pageTranslation.ts` detects page language, authorizes
  browser-UI requests, and runs bounded code in a dedicated isolated world.
- `electron/main/onDeviceTranslation.ts` owns the isolated inference worker,
  model cache, progress, and request lifetime.
- `ui/components/navoverlay/AppearanceSection.vue` owns profile configuration.
- `ui/components/PageTranslationBar.vue` owns the browser-controlled offer and
  result surface.
- `packages/captainword` maps profile languages onto the language packs that
  Captain Word actually supports.

## Translation model attribution

The multilingual model is
[`facebook/m2m100_418M`](https://huggingface.co/facebook/m2m100_418M), converted
to ONNX by Xenova for Transformers.js. M2M100 was developed by Meta AI and is
licensed under the MIT license. Summer pins the ONNX repository revision
`9c374f0b7aca709787cea97b047bfbbd1559d177`.

The Korean-to-English fast path is
[`Helsinki-NLP/opus-mt-ko-en`](https://huggingface.co/Helsinki-NLP/opus-mt-ko-en),
converted to ONNX by Xenova for Transformers.js and licensed under Apache 2.0.
Summer pins ONNX revision
`d9fa1ac6008242100fecb4fff9b0a5917a1be81f`.

The Russian-to-English fast path is
[`Helsinki-NLP/opus-mt-ru-en`](https://huggingface.co/Helsinki-NLP/opus-mt-ru-en),
converted to ONNX by Xenova for Transformers.js and licensed under CC BY 4.0.
Summer pins ONNX revision
`afe8c6c738ec81b6d033fd8f44f9678a639a7c67`.
