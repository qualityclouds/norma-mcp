# Using Norma in Lovable

Lovable is very good at getting you to a working app. It is not trying to tell you whether that app is safe to put in front of real users, or whether an engineering team could take it over in six months.

Norma does that. It connects to Lovable as a custom MCP chat connector, checks every file against your engineering standards as the code is written, then keeps a record of what it enforced. No plugin or partnership needed: Lovable already supports custom MCP servers.

Lovable generates React, Vite, TypeScript and Supabase. Norma ships 94 rules for exactly that stack. A first check on a real project tends to surface things like a Supabase service-role key in the browser bundle (the failure class behind CVE-2025-48757, which exposed data in over 170 Lovable-built apps), auth guards with inverted logic, `useEffect` loops that freeze the tab, no error boundary anywhere, and a missing Content Security Policy that blocks SOC 2 sign-off.

**Free tier is permanent.** Unlimited enforcement in the editor, no trial clock.

## Setup

### 1. Add Norma as a connector

In your Lovable project, open the connectors panel and connect a custom MCP server:

| Field | Value |
|---|---|
| Name | `Norma` |
| Server URL | `https://api.qualityclouds.ai/mcp` |
| Transport | HTTP |
| Auth | OAuth. Your browser opens to sign in on first connection |

No account yet? Sign up at [norma.qualityclouds.com](https://norma.qualityclouds.com). Free, no card.

Norma is a chat connector: it gives the Lovable agent context while it builds, and it is not bundled into your published app. It is personal to you.

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

**Lovable's internal git remote is a tokenised URL that it will not send to a third party**, and rightly so. The consequence is that `register_applied_actions` fails with *"link a repository first"* until you hand Lovable the public URL yourself. Your live checks work fine without this, but you get no audit trail, which is half the point of using Norma.

Fix it in one message. Paste your repository URL:

> this is my github repo for this app: https://github.com/yourname/yourrepo

Lovable calls `link_repository` and confirms the repository and branch. Audit registration works from then on, and `get_open_issues` starts returning the standing violations from your last full scan.

Do this at the start of the project, not after the first task, or the first task's evidence is lost.

No GitHub connection at all yet? Everything except `get_open_issues` and the audit trail still works. Norma does not require a repository.

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

**Symptom.** Live checks work. Then the audit registration fails, and Lovable tells you it needs the repository's public URL because its own remote is a tokenised internal one it will not send.

**Fix.** Paste your GitHub URL in chat and ask it to link. One message, once per project. Do it at the start, because evidence from tasks you ran before linking is not recoverable.

### Keep the audit trail honest

**Symptom.** A file comes back clean, and the agent then hunts for a rule id to attach the record to, settling on one that had nothing to do with the edit.

**Why it matters.** Entries claiming a rule was verified when it was never relevant are worse than no entry, because they make the record untrustworthy exactly where you need it to hold up.

**Fix.** Include the instruction from step 2: *only reference rules you actually evaluated.* If you are reviewing an audit trail and see the same generic rule id attached to unrelated changes, that is this problem, not a real pattern in your code.

### Large rulesets get truncated

**Symptom.** The agent notes that a tool result was truncated, usually after `get_rules_for_ruleset` on one of the bigger rulesets. Lovable caps how much tool output it will hold in context.

**Fix.** Do not ask it to pull every ruleset up front. Let `live_check` do the finding, since findings arrive with their own rule metadata attached, and only pull a full ruleset when you actually want to read the standards. If you need the rules in context for a specific piece of work, name the one ruleset you care about rather than asking for all of them.

## Good to know

- **Deterministic.** Norma runs Semgrep, regex and static architectural analysis. Same file, same rules, same verdict, every time. That is why the audit record it produces is worth something.
- **Your code is never stored.** `live_check` holds file content in memory for the duration of the check and nothing else.
- **Norma never writes to your repository.** It reports and records; Lovable applies the fixes, and you review them like any other change.
- **Same rules everywhere.** Cursor, Claude Code, VS Code, Windsurf, Replit, and any MCP-capable client. A project does not change grade when it moves tools.

Questions and support: [github.com/qualityclouds/community](https://github.com/qualityclouds/community/discussions)
