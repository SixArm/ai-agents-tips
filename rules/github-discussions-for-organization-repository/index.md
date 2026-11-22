# GitHub discussions for organization repository

Provide GitHub discussions for an organization repository

## Test

The program `bin/test-github-discussions` runs the tests.

Verify each file exists:

- locales/en-us/communications/github-discussions/announcements.txt: Announcements: Updates from maintainers
- locales/en-us/communications/github-discussions/general.txt: General: Chat about anything and everything here
- locales/en-us/communications/github-discussions/ideas.txt: Ideas: Share ideas for new features
- locales/en-us/communications/github-discussions/polls.txt: Polls: Take a vote from the community
- locales/en-us/communications/github-discussions/qa.txt: Q&A: Ask the community for help
- locales/en-us/communications/github-discussions/show-and-tell.txt: Show and tell: Show off something you've made

Verify each link works:

- https://github.com/orgs/[slug]/discussions must contain "View All Discussions"
- https://github.com/[slug]/[slug]/discussions must redirect to https://github.com/orgs/[slug]/discussions

### make

Run `make test` which depends on `make test-github-discussions`

Run `make test-github-discussions` which runs `bin/test-github-discussions`
