# llms.json and llms.txt

Goal: Provide AI tools with a clean, curated map of its most important content. Help large language models (LLMs) read, understand, and cite a site's documentation or resources.

Create files at the repo workspace root:

- `llms.json` -> JSON
- `llms.txt` -> markdown text
- The workspace-root llms.txt/llms.json use repo-relative links (e.g. README.md), which only resolve inside the git checkout.

If there is a project <repo>.github.io that is GitHub Pages, then also create files in the GitHub pages project:

- `<reop>.github.io/static/llms.json` -> JSON
- `<repo>.github.io/static/llms.txt` -> markdown text
- Serving that exact text from *.github.io/llms.txt uses website-appropriate
  versions *.github.io/static/ instead — pointing each entry at wherever it
  actually resolves from the site's own domain.

File size: < 40k bytes.
