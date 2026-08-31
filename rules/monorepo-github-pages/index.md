# Monorepo GitHub pages

Goal: publish Monorope GitHub pages by using the monorepo git subtree to export a sibling read-only repo.

This project is a monorepo: `~/git/<organization>/<repo>`

This project contains a GitHub pages subproject: `~/git/<organization>/<repo>/<repo>.github.io`

The GitHub pages subproject uses:

- [GitHub Pages](https://pages.github.com/)
- [SvelteKit](https://svelte.dev/docs/kit/)
- [Lily Design System](https://github.com/LilyDesignSystem/lily-design-system)

## Publish

To publish the GitHub pages subproject, use git subtree to derive a sibling top-level read-only export project: `~/git/<organization>/<repo>.github.io`

## Maintenance

Always maintain the GitHub pages subproject: `~/git/<organization>/<repo>/<repo>.github.io`

To maintain the sibling top-level read-only export project, always use git subtree; never work directly in: `~/git/<organization>/<repo>.github.io`
