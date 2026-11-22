# Special files for public repos

Special files that use top-level markdown:

| File                                               | Purpose                                                                                                                                                              |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| README.md                                          | Compehensive introduction to the project. Tone is welcomeing, friendly, good for novices                                                                             |
| LICENSE.md                                         | SPDX license information                                                                                                                                             |
| LICENSE/*.md files                                 | One file per license with full text                                                                                                                                  |
| CITATION.cff                                       | ORCID citation for Joel Parker Henderson (joel@joelparkerhenderson.com) (see ~/git/assertables/assertiables/CITATION.md for template)                                |
| INSTALL.md                                         | how to install and use any of the software                                                                                                                           |
| CONTRIBUTING.md                                    | how a person can contribute their time, or update code, or donate money, etc.                                                                                        |
| CODEOWNERS                                         | GitHub codeowners file; joel@joelparkerhenderson.com                                                                                                                 |
| MAINTAINERS.md                                     | Joel Parker Henderson (joel@joelparkerhenderson.com) as sole maintainer (use this as template: https://github.com/rubentalstra/FerroEHR/blob/develop/MAINTAINERS.md) |
| CHANGELOG.md                                       | change log history summries                                                                                                                                          |
| docs/ai-statement/index.md                         | use this as template: https://github.com/rubentalstra/FerroEHR/blob/develop/AI_STATEMENT.md)                                                                         |
| docs/benchmarks/index.md                           | benchmarks, speed tests, optimizaiton profiles, etc.                                                                                                                 |
| docs/comparisons/index.md                          | comparisons to relevant projects, context, etc.                                                                                                                      |
| docs/coordinated-vulnerability-disclosure/index.md | coordinated vulnerability disclosre (see https://github.com/coordinated-vulnerability-disclosure)                                                                    |
| docs/news/index.md                                 | news, update information, press contacts, etc.                                                                                                                       |
| docs/rfc/index.md                                  | request for comments e.g. what does this project want to learn, what kinds of constructive feedback can help, how to improve, etc.                                   |
| docs/privacy-policy/index.md                       | privacy policy                                                                                                                                                       |
| docs/security/index.md                             | security information                                                                                                                                                 | request for comments e.g. what does this project want to learn, what kinds of constructive feedback can help, how to improve, etc. |

If any of these top-level files exist then move them:

| Old             | New                                               |
| --------------- | ------------------------------------------------- |
| NEWS.md         | docs/news/index.md                                |
| COMPARISONS.md  | docs/comparisons/index.md                         |
| BENCHMARKS.md   | docs/benchmarks/index.md                          |
| RFC.md          | docs/rfc/index.md                                 |
| FAQ.md          | docs/faq/index.md                                 |
| AI_STATEMENT.md | docs/ai-statement/index.md (symlink to README.md) |
| GOVERNANCE.md   | docs/governance/index.md                          |
| SECURITY.md     | docs/security/index.md                            |
