# Oxford spelling

This guidebook is written in **Oxford spelling** (also called Oxford English
Dictionary spelling, or the `-ize` convention). This page is the reference the
chapters are written to, and the spelling sweep in the toolchain enforces it.

Oxford spelling is British English with one deliberate difference: it uses
`-ize` and `-ization` where mainstream British writing often uses `-ise` and
`-isation`. It reflects the Greek suffix `-ίζω` (*-ízō*), the origin of that
verb ending. It is the house style of Oxford University Press and Cambridge
University Press, the journal *Nature*, and, most relevant here, the style
standard of the international organizations of the United Nations System.

The IETF language tag for this variety is **`en-GB-oxendict`** (the older
`en-GB-oed` was deprecated in 2015).

## Why we use it

Oxford spelling is the official or de facto standard in the style guides of the
international organizations that belong to the United Nations System. This
includes the World Health Organization (WHO), the International
Telecommunication Union (ITU), the International Labour Organization (ILO), the
World Food Programme (WFP), the International Court of Justice (ICJ), and
UNESCO, and it is used in all UN treaties and declarations, such as the
Universal Declaration of Human Rights.

Other international organizations that adhere to this standard include the
International Organization for Standardization (ISO), the International
Electrotechnical Commission (IEC), the World Trade Organization (WTO), the North
Atlantic Treaty Organization (NATO), the International Atomic Energy Agency
(IAEA), Interpol, the International Committee of the Red Cross (ICRC), the World
Wide Fund for Nature (WWF), Amnesty International, the World Economic Forum
(WEF), and the Global Biodiversity Information Facility (GBIF).

For a guide aimed at enterprise and government readers, matching the spelling of
international standards bodies and UN institutions is a deliberate fit.

## The core rule: `-ize`, not `-ise`

Use `-ize` and `-ization` for the roughly 200 verbs whose ending derives from
Greek `-ízō`, and for their derivatives.

| Use (Oxford) | Not |
| --- | --- |
| organize, organization | organise, organisation |
| realize | realise |
| recognize, recognizable | recognise, recognisable |
| standardize | standardise |
| prioritize | prioritise |
| optimize, optimization | optimise, optimisation |
| centralize, decentralize | centralise, decentralise |
| categorize | categorise |
| emphasize | emphasise |
| minimize, maximize | minimise, maximise |
| summarize | summarise |
| characterize | characterise |
| capitalize | capitalise |
| specialize, generalize | specialise, generalise |
| utilize, visualize, normalize | utilise, visualise, normalise |
| apologize, criticize | apologise, criticise |
| analog **but** `-ize` verbs above | (n/a) |

Nouns and adjectives follow: *organization*, *prioritization*,
*standardization*, *optimization*, *utilization*.

## The exception: words that keep `-ise` or `-yse`

Many words end in `-ise` where that ending is **not** the Greek suffix but part
of an English or French root. These keep `-ise` in every variety of English,
Oxford included. Do not "correct" them to `-ize`:

> advertise, advise, arise, chastise, comprise, compromise, demise, despise,
> devise, disguise, enterprise, excise, exercise, franchise, improvise,
> incise, merchandise, premise, prise (to force open), promise, revise,
> supervise, surmise, surprise, televise

Likewise, words from Greek *lysis* ("release") keep `-yse` in Oxford and British
English, unlike American English:

> analyse, catalyse, dialyse, electrolyse, hydrolyse, paralyse

(So: *analyse*, *paralyse*, but *analysis*, *catalyst* are unchanged.)

## British spellings that Oxford keeps

Oxford spelling is otherwise fully British. Keep these forms (and convert any
American spelling in the source back to them):

| Feature | Oxford / British | Not (American) |
| --- | --- | --- |
| `-our` | colour, behaviour, favour, labour, honour, neighbour, rumour, vapour, endeavour | color, behavior, favor, labor, ... |
| `-re` | centre, metre, theatre, litre, fibre, calibre, spectre | center, meter, theater, ... |
| `-ce` noun vs `-se` verb | licence (n) / license (v); practice (n) / practise (v); defence, offence, pretence | license (n), practice (v), defense |
| doubled `-ll-` | travelled, modelling, labelled, cancelled, signalling, counsellor, marvellous | traveled, modeling, labeled, ... |
| single `-l` | enrol, fulfil, instil, skilful, wilful, distil | enroll, fulfill, ... |
| `-ogue` | catalogue, dialogue, analogue | catalog, dialog, analog |
| `-mme` | programme (a scheme or broadcast) | program |
| `ae` / `oe` | encyclopaedia, paediatric, manoeuvre, oestrogen | encyclopedia, pediatric, ... |
| miscellaneous | grey, mould, smoulder, plough, sceptical, cheque, kerb, tyre, artefact, aluminium, whilst | gray, mold, skeptical, check, ... |

**Software exception.** A computer *program* is spelled *program* even in
British and Oxford usage. Reserve *programme* for a plan, agenda, or broadcast.
Keep established technical and product terms as their owners spell them (for
example `Kubernetes`, `TypeScript`, `color` in a CSS property or code sample).

**Judgement.** Oxford and general British usage prefer *judgement*; use
*judgment* only in the strict legal sense (a court's judgment). Similarly
*acknowledgement*.

## Rules for the spelling sweep

When converting a file to Oxford spelling:

1. **Never modify a URL.** Inside a Markdown link `[text](url)`, leave the `url`
   byte for byte unchanged. Wikipedia article slugs (for example
   `.../wiki/Internationalization`) are case- and spelling-sensitive; corrupting
   them breaks the link. You may Oxford-spell the visible link *text*, but if
   there is any doubt, leave the whole link untouched.
2. **Do not touch code.** Leave inline code spans, fenced code blocks, file
   names, identifiers, CSS properties, and CLI flags exactly as written.
3. **Do not alter proper nouns or titles.** Organization names, product names,
   and the exact titles of books, papers, and standards in reference lists stay
   as their owners spell them (for example the *International Organization for
   Standardization*).
4. **Preserve meaning-bearing distinctions.** Get the noun/verb pairs right:
   *licence*/*license*, *practice*/*practise*.
5. **Spelling only.** The sweep changes spelling, never structure, wording, or
   meaning.
