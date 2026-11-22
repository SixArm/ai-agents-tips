# Monorepo GitHub pages

Goal: publish Monorope GitHub pages by using the monorepo git subtree to export a sibling read-only repo.

This project is a monorepo: `~/git/<organization>/<repo>`

This project contains a GitHub pages subproject: `~/git/<organization>/<repo>/<repo>.github.io`

The GitHub pages subproject uses:

- [GitHub Pages](https://pages.github.com/)
- [SvelteKit](https://svelte.dev/docs/kit/)
- [Lily Design System](https://github.com/LilyDesignSystem/lily-design-system)

## Git config

The file `.git/config` must have a remote name github-page:

```toml
[remote "github-pages"]
        url = git@github.com:<organization>/<orgnization>.github.io.git
        fetch = +refs/heads/*:refs/remotes/github-pages/*
```

## Publish

To publish the GitHub pages subproject, use git subtree to derive a sibling top-level read-only export project: `~/git/<organization>/<repo>.github.io`

File `Makefile` provides task `make github-pages` that delegates to POSIX shell script `bin/make-github-pages` that runs `git subtree push --prefix=<name>.github.io github-pages main`

This pushes a subdirectory of your current repo out to a different branch (here main) on the remote named github-pages, using Git's subtree mechanism.

## Maintenance

Always maintain the GitHub pages subproject: `~/git/<organization>/<repo>/<repo>.github.io`

To maintain the sibling top-level read-only export project, always use git subtree; never work directly in: `~/git/<organization>/<repo>.github.io`
