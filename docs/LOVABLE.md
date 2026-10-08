# Using Norma in Lovable

Lovable is very good at getting you to a working app. It is not trying to tell you whether that app is safe to put in front of real users, or whether an engineering team could take it over in six months.

Norma does that. It connects to Lovable as a custom MCP chat connector, checks every file against your engineering standards as the code is written, then keeps a record of what it enforced. No plugin or partnership needed: Lovable already supports custom MCP servers on every plan.

Lovable generates React, Vite, TypeScript and Supabase. Norma ships 83 rules for exactly that stack, 47 of them HIGH severity. A first check on a real project tends to surface a Supabase service-role key compiled into the browser bundle, an admin client instantiated in a frontend component, auth guards with the condition inverted, `useEffect` loops that freeze the tab, no error boundary anywhere, and a missing Content Security Policy that blocks SOC 2 sign-off.

**Where this sits next to Lovable's own scans.** Lovable's Basic Scan runs on publish and lints your Row Level Security policies, your schema and your dependencies. That is the database side of the failure class behind CVE-2025-48757, and it is the right place to check it. Norma does not look at RLS policies or schema state. It looks at code, which is where the same data leaks with the policies intact: a service-role key in the bundle bypasses every policy you wrote. The two do not overlap, and you want both.

**Free tier is permanent.** Not a trial, no clock. The SOLID architecture ruleset is a Pro feature; everything above is on Free.

## Setup

### 1. Add Norma as a connector

In your Lovable project, open **Connectors**, go to the **All** view, scroll to the bottom, and choose the **Custom** card labeled **MCP** ("Connect your own MCP").

| Field | Value |
|---|---|
| Server name | `Norma` |
| Server URL | `https://api.qualityclouds.ai/mcp` |
| Authentication | **OAuth**. This is Lovable's default, so leave it as is |

Click **Add & authorize**. Your browser opens to sign in.

No account yet? Sign up at [norma.qualityclouds.com](https://norma.qualityclouds.com/?utm_source=github&utm_medium=readme&utm_campaign=norma-mcp&utm_content=lovable-guide). Free, no card.

Norma is a chat connector: it gives the Lovable agent context while it builds, and it is not bundled into your published app. Chat connections are per user, so a teammate on the same project connects their own.

### 2. Say this once

You do not need a long system prompt. Send this as a normal message and Lovable saves it as a standing rule for the project:

> From now on, for every file you create or change, run it through Norma and fix the issues it returns.

Lovable will confirm something like *"Saved as a standing rule: every file I create or change goes through Norma's live check, I fix what it flags, re-check, and register the outcome in your audit trail at the end of each task."* From that point on it applies to every message. You never mention Norma again.

**If you want it stricter,** add these two lines. They close the one gap in the short version:

> Never tell me the code is clean unless live_check actually returned clean.
> "not_checkable" and "refused" are not clean.
>
> When you register the outcome, only reference rules you actually evaluated.
> If a file came back clean with nothing specific to report, say so rather
> than picking a rule id to attach it to.

That second instruction matters more than it looks. See *Keep the audit trail honest* below.

### 3. Link your repository. This step is not optional.

**Lovable's internal git remote is a tokenized URL that it will not send to a third party**, and rightly so. Live checks only run on a repository linked in your Norma account, so this takes two steps.

First, link the project's GitHub repository at [norma.qualityclouds.com](https://norma.qualityclouds.com/?utm_source=github&utm_medium=readme&utm_campaign=norma-mcp&utm_content=lovable-guide) through the GitHub integration. Your code is never stored in your Norma account: linking tells Norma which repository a check belongs to, and nothing more.

Second, give Lovable the public URL, so it can tell Norma which repository it is working on. Until it has it, `live_check` and `register_applied_actions` fail with *"link a repository first"*, and Norma checks nothing and records nothing.

Fix it in one message. Paste your repository URL:

> this is my github repo for this app: https://github.com/yourname/yourrepo

Lovable confirms the repository. Live checks and audit registration work from then on, and `get_open_issues` starts returning the standing violations from your last full scan.

Do this at the start of the project, before the first task, or that task goes unchecked.

No GitHub connection at all yet? Connect the Lovable project to GitHub first, then link that repository in Norma. Live checks need a linked repository.

## What changes in your build loop

Nothing slows down. What changes is that the agent stops handing you work it has not checked.

Here is a real exchange from a Lovable project with the standing rule in place. The request has nothing to do with code quality; it is a straightforward feature removal.

> **You:** take out the Company Watch list feature.
>
> **Lovable:** Company Watchlist is gone: removed the dashboard card (`WatchlistCard` deleted), stripped the watch/unwatch column and state from the Visitor IPs table, and Live Visitors now spans full width. IP tagging is untouched.
>
> Norma check on the changed files came back clean after I fixed 3 findings it flagged in the surrounding code (setState directly in `useEffect`, three unguarded Supabase awaits, and an async `forEach`).

Nobody asked for a code review. The agent touched three files, ran them through Norma, found three pre-existing problems in the code around its edit, fixed them, and re-checked. None of those would have been caught by asking "is this feature removed properly?" All three were going to production.

### Auditing what you already built

If you have an existing Lovable project, start here instead:

> Call Norma's get_rulesets, pull the rules for each, then live_check every
> file under src/. List everything HIGH severity first, and fix them in
> severity order. Confirm each fix with live_check before moving on.

Expect the first run to be uncomfortable. That is the point of running it.

## Troubleshooting

### "Link a repository first"

**Symptom.** The first live check or audit registration fails, and Lovable tells you it needs the repository's public URL because its own remote is a tokenized internal one it will not send.

**Fix.** Check that the repository is linked in your Norma account, then paste its public GitHub URL in chat. One message, once per project. Do it at the start, because files written before linking were never checked.

### Keep the audit trail honest

**Symptom.** A file comes back clean, and the agent then hunts for a rule id to attach the record to, settling on one that had nothing to do with the edit.

**Why it matters.** Entries claiming a rule was verified when it was never relevant are worse than no entry, because they make the record untrustworthy exactly where you need it to hold up.

**Fix.** Include the instruction from step 2: *only reference rules you actually evaluated.* If you are reviewing an audit trail and see the same generic rule id attached to unrelated changes, that is this problem, not a real pattern in your code.

### Large rulesets get truncated

**Symptom.** The agent notes that a tool result was truncated, usually after `get_rules_for_ruleset` on one of the bigger rulesets. Lovable caps how much tool output it will hold in context.

**Fix.** Do not ask it to pull every ruleset up front. Let `live_check` do the finding, since findings arrive with their own rule metadata attached, and only pull a full ruleset when you actually want to read the standards. If you need the rules in context for a specific piece of work, name the one ruleset you care about rather than asking for all of them.

### `get_open_issues` looks incomplete

**Symptom.** The repository has more open issues than the agent reports.

**Fix.** Called with no arguments, `get_open_issues` returns the top 25 by severity, with a total count and a portal link. Ask for a specific set of files and you get the complete list for those files.

## Good to know

- **Deterministic.** Norma runs Semgrep, regex and static architectural analysis. Same file, same rules, same verdict, every time. That is why the audit record it produces is worth something.
- **Your code is never stored.** `live_check` holds file content in memory for the duration of the check and nothing else.
- **Norma never writes to your repository.** It reports and records; Lovable applies the fixes, and you review them like any other change.
- **Same rules everywhere.** Cursor, Claude Code, VS Code, Windsurf, Replit, and any MCP-capable client. A project does not change grade when it moves tools.

Questions and support: [github.com/qualityclouds/community](https://github.com/qualityclouds/community/discussions)
