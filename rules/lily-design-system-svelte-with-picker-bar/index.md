# Lily Design System Svelte with PickerBar

For each Svelte app subdirectory...

Use PNPM and Lily dependencies (not vendored):

- https://www.npmjs.com/package/@lilydesignsystem/svelte-headless
- https://www.npmjs.com/package/@lilydesignsystem/svelte-theme-picker
- https://www.npmjs.com/package/@lilydesignsystem/svelte-text-size-picker
- https://www.npmjs.com/package/@lilydesignsystem/svelte-locale-picker
- https://www.npmjs.com/package/@lilydesignsystem/svelte-share-picker
- https://www.npmjs.com/package/@lilydesignsystem/svelte-picker-bar

In global top header navigation area use:

- svelte-picker-bar

## ThemePicker

- Use all Lily default themes (not any application-specific custom themes)
- Sort themes alphabetetically
- Sort UK & US themes at the end, after all the non-national themes

## Lily LocalePicker

- Sort labels alphabetically by locale code.
- Locale codes *-001: format label as "<language>" only NOT language region.
- Other locale codes: format label as "<language> - <region>" NOT "<language> (<region>)".
- Show locale cy-001; omit locale cy-gb.

## Lily SharePicker

If navigator.share is available then use it.

Otherwise use custom code with these choices:

- Copy Link
- Email Link
- Share on LinkedIn
- Share on Reddit
- Share on Bluesky
- Share on Mastodon (link to mastodonshare.com)

Double-check SharePicker button vertical align with other buttons (because we've seen so many bugs with this).

## Lily TextSizePicker

Use all Lily default text sizes

NO any application-specific custom text sizes.

## Update

Retire:

- If any Lily Design System components are vendored, such as in `./src/lib`, and are unneeded, then delete them.
- If any Lily custom CSS themes exist (not Lily theme defaults), then delete them.

After Lily updates:

1. commit, merge into main, delete old branches
2. publish to GitHub pages

## Verify

Verify all PickerButtons work.

When a user clicks a button:

- the dropdown list shows
- the dropdown list is wide enough to show all text in it
- the dropdown list does NOT cause the page to jump

Verify public GitHub pages site uses PickerBar:

- All PickerButtons work correctly
- ThemePicker has all Lily themes, and correct sort order
- LocalePicker has all Lily themes, and correct sort order, and no parentheses
- TextSizePicer has all Lily text sizes
- SharePicker works
