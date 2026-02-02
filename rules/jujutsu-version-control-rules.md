  # Version Control

  - IMPORTANT: YOU MUST assume we use Jujutsu ("jj") for version control. Check by running `jj root` (or look for a  `.jj/` directory) to confirm.
  - If I say "commit", "branch", "squash", etc., assume jj equivalents first. 
  - You can ignore any `git add` command, since jj always auto-adds.
  - Use `jj` commands instead of `git`. Never run `git` in a jj repository.
  - ALWAYS prefer jj change IDs over commit SHAs. jj change IDs are stable.
  - jj doesn't usually require named branches
  - If you need named branches, use `jj bookmark`, but you must manually update the bookmark after making new commits, since they won't automatically get updated
  - When reading data from jj, always use `--ignore-working-copy` to avoid snapshotting the working copy (which is slow and unnecessary for read operations). But when writing (commit, squash, rebase, etc.), you MUST NOT use `--ignore-working-copy`.
  - If you get "Error: The working copy is stale", run `jj workspace update-stale` first
  