# Locales for major projects with SvelteKit

Translate content into multiple locales.

How this site supports multiple locales end to end: content, web
routing, UI chrome, and bugs.

Read locales via file `locales.tsv`.

Locale code priority order:

- en-001
- en-us
- en-gb
- en-gb-oxendict
- cy-gb (then copy to cy-001)
- zh-cn (then copy to zh-001)
- ar-001
- hi-in (then copy to hi-001)

## .locale-peer.id file

`.locale-peer-id` file is a byte-identical 32-character hexadecimal lowercase
number then newline, across every locale's version of "the same" topic,
regardless of slug.

`.locale-peer-id` id is how the project resolves "this page, in locale X".

## Guidance

- en-us: consistent American spelling; fix any stray en-gb forms (organisation→organization, licence→license, programme→program, cancelled→canceled, analogue→analog).

- en-gb: the -ize/-ise family (optimise, realise, organise, prioritise, utilise, etc.), -or/-our (colour, behaviour, favour, labour, neighbours), -er/-re (centre, theatre for the metaphorical sense), -ense/-ce (defence, licence), doubled-L forms (modelled, labelled, cancelled, enrol/enrolment), analogue, programme, and math→maths.

- en-gb-oxendict: use en-gb then revert just the -ise family back to Oxford -ize spelling (optimize, realise→realize, organise→organize, etc.), while correctly keeping -yse forms (analyse/analysable) unchanged, since Oxford style never uses -yze, and keeping all other British forms (colour, centre, defence, licence, programme, maths, modelled) intact.

## Guard against corruption

Keep proper nouns unconverted. Example: "Hospital Readmissions Reduction Program" (a real United States federal program name).

## Verify

For each locale subdirectory:

- File exists: `index.md`
- Symlink exists: `README.md`
- Locale peer id tracking file exists: `.locale-peer-id`

Then:

- Fix any broken internal links
- Fix any residual wrong-dialect spellings
- Update `./spec/locale/index.md`

## Content structure (book side)

Each locale is `locales/<code>/` in the book repo, containing:

- `locales/<code>/topics/<slug>/index.md` + `.locale-peer-id` — one per topic.
  `README.md` is a symlink to `index.md`.
- `locales/<code>/index.md` + `.locale-peer-id` + `README.md` symlink — the
  locale's own translated README (site home/contents page source). Every
  locale gets this file scaffolded (matching the topic-file pattern) even
  before it has a translation; it starts empty.

## Slugs

Slugs are per-locale, not shared.** Translated locales rename topic directories
to native-script/accented slugs.

Example: `es-001` `año-de-vida-ajustado-por-calidad`, `ur-001` `صحت-ایڈجسٹڈ-متوقع-زندگی`.

Nothing in the site assumes slugs match across locales.

## Locale picker (labels + ordering)

- Labels live in `locales.js`'s `LOCALE_LABELS`, one entry per code, in that
  language (e.g. `'fr-001': 'Français (Monde)'`). Falls back to the raw code
  via `localeLabel()` if a code has no label yet.
- Header `PickerBar` order comes from `content.js`'s `locales()` (sorted by
  code) — the `-001` suffix happens to sort before any letter-starting
  regional suffix, so variants already come first there.
- Home page's locale list (`+page.server.js`) sorts explicitly: default
  locale first, then grouped by language name (label text before the `(`),
  with the `-001`/World variant sorted before its regional siblings within
  each group, then alphabetically by label. This does NOT fall out of
  alphabetical-by-label sort on its own (e.g. "España" < "Mundo") — it needs
  the explicit `-001` check.

## Bug fixes (regression watch-list)

### Bug: ASCII-only `\w` regexes broke every non-Latin/non-accented slug

Bug: matched topic slugs with `[\w.-]+` (ASCII word chars only). Any locale with
an accented or native-script slug (Spanish, French, Russian, Chinese, Arabic,
Welsh, Hindi, Bengali, Portuguese, Indonesian, Urdu) silently failed peer-id
resolution and cross-topic links.

Fix by widening the slug capture group to `[^/]+`.

### Bug: Every locale's home/contents page showed canonical English content

Bug: code and content always read a single top-level `/README.md` for title,
intro, "New here?" picks, part headings, and blurbs — only topic _links_ were
ever localized.

Fix: populate the previously-empty `locales/<code>/index.md` per locale.

## Bug: Link extraction was hardcoded to literal English phrase

Bug: link silently found nothing once the README was translated.

Fix: extract all links from the whole pre-`##` intro block instead of
regex-matching the English sentence.

### Bug: UI chrome was hardcoded English in the `.svelte` templates

Bug: nav labels, subtitles, page titles, intros, breadcrumbs, topic position,
pagination, picker/share labels.

Fix: add `i18n.js` and threading `ui(locale)` through every locale-scoped route
and `+layout.svelte`.

### Bug: header/footer brand wordmark stayed English

Bug: wordmark came only from the root (locale-agnostic) `+layout.server.js`,
which deliberately never picks a locale.

Fix: have `locales/[locale]/+layout.server.js` supply this locale's own title,
which overrides the root layout's canonical one via SvelteKit's merged
`page.data` on any route under `/locales/<locale>/` — the root picker and
`/about/` (no locale in the URL) correctly keep the canonical English title.

## Defer to do manually

Goal: translate our projects into locales; work serially NOT subagents.

Verify project website "https://<project>.github.io/" works and page has translations (and fix any missing or errors):

- hero headline
- hero subheadline
- hero one-paragraph introduction
- if the project is lily-design-system then show icons of frameworks
- hero one-sentence "Translations: " with comma list of *-001 translation locale endodyms; example using locale en-001: "Translations: العربية, বাংলা, Cymraeg, Deutsch, English, Español, Français, हिन्दी, Bahasa Indonesia, 日本語, 한국어, Português, Русский, اردو, 中文"
- each pickerbar button: hover changes the color
- each pickerbar button: dropdown appears under the button, and does not shift the page
- has file `https://<project>.github.io/sitemap.xml` that is a site map XML of the website

For each locale in order:

- en-001
- en-us
- en-gb
- zh-cn then copy to zh-001
- hi-in then copy to hi-001
- cy-gb then copy to cy-001
- ar-001
- es-001
- fr-001
- ru-001
- bn-001
- id-001
- ja-jp then copy to ja-001
- ko-kr then copy to ko-001
- sw-tz then copy to sw-001
- sv-se then copy to sv-001
- nl-nl then copy to nl-001
- is-is then copy to is-001

Verify project locale works (and fix any missing or errors)

- has file `/locales/<locale>/sitemap.xml` that is a sitemap of the subdirectory routes
- has all its content files
- each `/locales/<locale>/` subdirectory contains tracking file `.locale-peer-id`
- each `/locales/<locale>/` subdirectory segment is translated into language and character set (not merely romanized)

If project is missing a locale:

- create the locale and translate the content serially NOT subagents.
- commit and push and publish
- verify browser "https://<project>.github.io/<locale>/" successfully renders the locale.

Verify project locale website "https://<project>.github.io/<locale>/" works and page has translations (and fix any missing or errors):

- hero headline
- hero subheadline
- hero one-paragraph introduction
- hero one-sentence "Translations: " with comma list of translation locale endodyms; example using locale en-001: "Translations: العربية, বাংলা, Cymraeg, Deutsch, English, Español, Français, हिन्दी, Bahasa Indonesia, 日本語, 한국어, Português, Русский, اردو, 中文"

Continue until all complete.

## Stop

More locales:

- de-001
- pt-001
- ur-001
- bn-001
- da-001
- et-ee then copy to et-001
- id-id then copy to id-001
- fi-fi then copy to fi-001
- it-it then copy to it-001
- th-th then copy to th-001
- tr-tr then copy to tr-001
- vi-vn then copy to vi-001
- zh-tw

## Locale choice reasoning

Where I live:

- en-gb English - Great Britain
- cy-gb Cymreag - Great Britain

Worldwide top 3 order by count:

- en-us English - United States
- zh-cn Chinese - China
- hi-in Hindi - India

United Nations working languages:

- en-us English - United States
- zh-cn Chinese - China
- es-es Spanish - Spain
- fr-fr French - France
- ar-eg Arabic - Egypt
- ru-ru Russian - Russia

Association of Southeast Asian Nations (ASEAN) working language:

- en-001 English

African Union working languages:

- ar-eg Arabic - Egypt
- en-us English - United States
- fr-fr French - France
- es-es Spanish - Spain
- pt-pt Portuguese - Portugal
- sw-tz Swahili - Tanzania

Worldwide by country population:

- id-id Indonesian - Indonesia
- pt-br Portuguese - Brazil
- es-mx Spanish - Mexico

Eurpean Union top 3:

- de-de German - Germany
- fr-fr French - France
- it-it Italian - Italy

APAC top 3:

- id-id Indonesian - Indonesia
- ja-jp Japanese - Japan
- vi-vn Vietnamese - Vietnam

Nordic top 3:

- sv-se Swedish - Sweden
- da-dk Danish - Denmark
- fi-fi Finnish - Finland

Public value top 3:

- zh-tw Chinese - Taiwan
- ko-kr Korean - South Korea
- nl-nl Dutch - Netherlands

Worldwide top 10 order by count:

- en-us English - United States
- zh-cn Chinese - China
- hi-in Hindi - India
- es-es Spanish - Spain
- fr-fr French - France
- ar-eg Arabic - Egypt
- bn-bd Bengali - Bangladesh
- ru-ru Russian - Russia
- pt-pt Portuguese - Portugal
- ur-pn Urdu - Pakistan

Public health government leadership recommendations:

- da-dk Danish - Denmark
- et-ee Estonian - Estonia

Nordic countries order by count:

- sv-se Swedish - Sweden
- da-dk Danish - Denmark
- fi-fi Finnish - Finland
- nb-no Norwegian - Norway
- is-is Icelandic - Iceland

Countries that welcomed me:

- vi-vn Vietnamese - Vietnam
- lo-la Laotian - Lao
- km-kh Cambodian - Cambodia
- th-th Thai - Thailand

Celtic top 3 order by proximity:

- cy-cy Cymraeg - Cymru
- gd-gb Scottish Gaelic - Scotland
- ga-ie Irish - Ireland

North America:

- en-us
- es-mx
- fr-ca

Eurpean Union order by speaker count:

- de-de German - Germany
- fr-fr France - French
- it-it Italian - Italy
- es-es Spanish - Spain
- pl-pl Polish - Poland
- ro-ro Romanian - Romania
- nl-nl Dutch - Netherlands
- cs-cz Czech - Czechia
- sv-se Sweden - Swedish
- pt-pt Portuguese - Portugal
- hu-hu Hungarian - Hungary
- de-at German - Austria
- el-gr Greek - Greece
- bg-bg Bulgarian - Bulgaria
- da-dk Denmark - Danish
- fi-fi Finnish - Finland
- sk-sk Slovak - Slovakia
- fr-be French - Belgium
- hr-hr Croatian - Croatia
- lt-lt Lithuanian - Lithuania
- sl-si Slovenian - Slovenia
- lv-lv Latvian - Latvia
- et-ee Estonian - Estonia
- el-cy Greek - Cyprus
- lb-lu Luxembourgish - Luxembourg
- mt-mt Maltese - Malta
- ga-ie Irish - Ireland

More languages with 50M+ speakers in order of count:

- pa-001 ਪੰਜਾਬੀ Punjabi
- mr-001 मराठी Marathi
- te-001 తెలుగు Telugu
- ha-001 Harshen Hausa Hausa
- ta-001 தமிழ் Tamil
- fa-001 فارسی Persian
- jv-001 Basa Jawa Javanese
- gu-001 ગુજરાતી Gujarati
- bh-001 भोजपुरी Bhojpuri

Supressed languages:

- cy-gb Cymraeg Welsh
- eu-001 Euskara Basque

TODO:

- hr-001 Hrvatski Croatian
- hu-001 Magyar Hungarian
- tr-001 Türkçe Turkish
