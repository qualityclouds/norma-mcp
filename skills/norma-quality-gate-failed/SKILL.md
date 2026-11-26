---
name: norma-quality-gate-failed
description: Fix a pull request whose "Norma Quality Gates" check failed. Use
  when the GitHub check or the Bitbucket build status "Norma Quality Gates" on
  your pull request fails, or when a Norma comment on the pull request reports
  blocking findings. It does not need the pull request comments.
---

# Fix a failed Norma Quality Gate

Norma evaluates each pull request against the organization's Quality Gates and
reports the result as "Norma Quality Gates": a check run on GitHub, a build
status and a Code Insights report on Bitbucket. A failed result blocks the
merge until the blocking findings are fixed.

## When to use

- The "Norma Quality Gates" check or build status on your pull request failed.
- A comment from Norma on your pull request lists blocking findings.

## Steps

1. Read the details of "Norma Quality Gates" if you can: the check run output
   and its annotations on GitHub, the Code Insights report on Bitbucket. They
   give the blocking findings with rule, file and line. If you cannot read
   them, continue with step 2.
2. List the files that the pull request adds or modifies
   (`git diff --name-only --diff-filter=AM origin/<base>...HEAD`). Read the
   remote once (`git remote get-url origin`) and call `live_check` with those
   files and their current content.
3. Fix the findings from steps 1 and 2. Call `live_check` again on the files you
   changed. Repeat until the result is clean.
4. Commit and push to the same branch. Norma evaluates the new commit and
   updates "Norma Quality Gates" on the same pull request.
5. Call `register_applied_actions`: the findings you fixed go in
   `violations_fixed`, with the file, the lines and the rule IDs.

## Rules

- Fix the code. Do not re-run the old check, do not open a new pull request,
  and do not ask for the gate to be bypassed.
- A gate finding that you cannot fix without a change the user did not ask
  for: leave it, and state the rule ID, the file and the reason in a pull
  request comment, so a person decides.
- If the gate still fails after the push and `live_check` is clean, use the
  findings in the new check details from step 1.
