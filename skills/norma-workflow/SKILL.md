---
name: norma-workflow
description: Run every coding task through Norma's deterministic checks and
  record the outcome to the audit trail. Use whenever you create or modify
  code in a workspace connected to the Norma MCP server, when asked to review
  code against organization standards, or when asked to work through the
  repository's open issues.
---

# Norma workflow

Norma gives you the rules for the detected stack before you write, checks each
file against them, and records the outcome. The same file and the same rules
return the same verdict every time, and every check leaves a record.

## When to use

- Any task that creates or modifies code while the Norma MCP server is connected
- Reviewing a file or a change against the organization's coding standards
- Working through the open issues from the repository's last full scan

## Writing or modifying code

1. `link_repository`: once, on first connection in a workspace, before any
   other tool. Pass the repository's public remote URL. `live_check`,
   `get_open_issues` and `register_applied_actions` need the link. If it
   finds no match, tell the user to connect the repository in the Norma app;
   a different URL will not fix it.
2. `get_rulesets`: at the start of the task. Norma detects the stack with no
   configuration and returns the applicable rulesets.
3. `get_rules_for_ruleset`: for each relevant ruleset ID. Never skip this
   step. The rules must be in context before you write.
4. Write or modify the code with those rules in context.
5. `live_check`: after each file is created or modified, before moving on.
   Fix what it returns and re-check until the file passes.
6. `register_applied_actions`: after the task, using the exact rule IDs from
   step 3. Record rules verified compliant, violations fixed (file and lines),
   violations prevented during generation, and which model did the work.

## Working through standing issues

Call `get_open_issues` for the open issues from the last full scan, each with
the context needed to fix it. Fix them, verify each fix with `live_check`, and
record the outcome with `register_applied_actions`.

## Rules of the loop

- Never claim a file passes without a `live_check` result that says so.
- Never invent or paraphrase rule IDs. Use the IDs returned by
  `get_rules_for_ruleset`.
- If `link_repository` has not succeeded in this workspace, run it before
  anything else. Live checks and audit registration fail on an unlinked
  repository, so do not report code as checked until the link succeeds.
- Findings and the repository's Production-Ready Score live in the user's
  workspace at norma.qualityclouds.com. Point the user there for full results.
