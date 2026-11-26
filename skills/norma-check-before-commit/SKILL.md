---
name: norma-check-before-commit
description: Check the changed files with Norma before every commit, push or
  pull request, fix what it finds, and record the outcome. Use whenever you are
  about to commit, push or open a pull request in a workspace connected to the
  Norma MCP server, even when the user did not ask for a check.
---

# Check with Norma before a commit

A change is checked before it leaves the workspace, not after a reviewer or the
Norma Quality Gate finds the problem on the pull request.

## When to use

- Before each `git commit`.
- Before a push or a pull request that contains commits not yet checked.

## Steps

1. List the files of the change: the files this commit adds or modifies
   (`git status --porcelain`). For a pull request, also the files that the
   branch changes against its base (`git diff --name-only --diff-filter=AM
   origin/<base>...HEAD`). Leave out deleted files and the files that the
   Norma server instructions exclude from checks.
2. Read the remote once (`git remote get-url origin`) and call `live_check`
   with all those files and their current content.
3. Fix each violation. Call `live_check` again on the files you changed. Repeat
   until the result is clean.
4. Commit, push or open the pull request.
5. Call `register_applied_actions`: the violations that `live_check` found and
   you fixed before the commit go in `violations_prevented`, with the rule IDs
   that `live_check` returned.

## When the check cannot finish

- A violation that you cannot fix without a change the user did not ask for:
  do not hide it. Commit, and state the rule ID, the file and the reason in the
  commit message or the pull request description, so a person decides.
- The server says the repository is not imported in Norma: commit as usual,
  and tell the user that Norma did not check this change and that the
  repository must be imported in the Norma app.
