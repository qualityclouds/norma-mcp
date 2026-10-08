<!-- mcp-name: ai.qualityclouds/norma -->
<p align="center"><img src="assets/logo.png" alt="Norma by Quality Clouds" width="120"></p>

# Norma by Quality Clouds: MCP Server

Norma by Quality Clouds checks AI-written code against your rules as it is written, and keeps a record of what it found.

Norma gives your coding agent the rules for your detected stack before it writes, checks each file against them, and records the outcome. The same file and the same rules return the same verdict every time, so what you get back is a record you can show someone rather than an opinion that changes between runs.

- Remote server (Streamable HTTP): `https://api.qualityclouds.ai/mcp`
- Auth: OAuth. Sign in via browser on first connection. Permanent free tier.
- Requires a linked repository: `live_check` only runs on a repository linked in your Norma account (GitHub or Bitbucket). Your code is never stored there
- Works with Cursor, Claude Code, Lovable, Replit and any MCP-compatible client
- Stacks covered: TypeScript, JavaScript, Python, PHP, Dart and Flutter, Node, React, Vite, Supabase, FastAPI, SQLAlchemy, Adobe Magento and more

## What it does

Rulesets are stack-specific and activate automatically for the stack the server detects. A single repository scan activates rulesets across six functional areas: Security, Performance, Scalability, Manageability, Maintainability and Architecture. The Supabase ruleset alone covers 12 rules, including no service-role keys outside the server, no hardcoded keys or project URLs, mandatory error checks on every mutation, no client-side JWT decoding, and rate limiting on edge functions.

The server exposes five tools:

| Tool | What it does |
|------|--------------|
| `get_rulesets` | Detects your repository's tech stack, with no configuration, and returns the applicable rulesets, one per language or framework |
| `get_rules_for_ruleset` | Returns the rules in one ruleset: each rule's ID, name, severity, impact area, description and fix guidance |
| `live_check` | Deterministic check of one file against your organization's rules. Same file, same rules, same verdict every time. Nothing is stored. Runs only on a repository linked in your Norma account |
| `get_open_issues` | Returns the open issues from the last full scan of the linked repository, each with the context needed to fix it |
| `register_applied_actions` | Records what was done in your compliance audit trail: rules verified compliant, violations fixed (file and lines), violations prevented during generation, and which model did the work |

Every session produces a structured record of what was checked, fixed and prevented. Your repository's Production-Ready Score and full findings live in your workspace at [norma.qualityclouds.com](https://norma.qualityclouds.com/?utm_source=github&utm_medium=readme&utm_campaign=norma-mcp&utm_content=workspace).

## How it works

Before the first check, link your repository in your Norma account (see the 5-minute guide below). After that, a coding agent connected to Norma follows this sequence:

1. `get_rulesets`: at the start of any coding task.
2. `get_rules_for_ruleset`: for each relevant ruleset ID; never skipped.
3. The agent writes or modifies code, with those rules in context.
4. `live_check`: after each file is created or modified, before moving on.
5. `register_applied_actions`: after the task, using the exact rule IDs from step 2.

To work through standing issues instead of writing new code, the agent calls `get_open_issues` and fixes the violations from the repository's last full scan.

## Try it in 5 minutes (Claude Code)

1. Link your repository in Norma. Sign in at [norma.qualityclouds.com](https://norma.qualityclouds.com/?utm_source=github&utm_medium=readme&utm_campaign=norma-mcp&utm_content=link-repo) (free to start, no credit card) and link the repository through the GitHub or Bitbucket integration. Live checks only run on a linked repository. Your code is never stored in your Norma account: linking tells Norma which repository a check belongs to, and `live_check` holds file content in memory only for the duration of the check.

2. Add the server:

```bash
claude mcp add --scope user --transport http norma https://api.qualityclouds.ai/mcp
```

3. Check it connected:

```bash
claude mcp list
```

4. On first use, your browser opens to sign in.
5. Open a session in your local clone of the repository you linked and ask Claude Code to review a file against your coding standards. It will fetch the rules for your stack, live check the file, and register the results back to your workspace.

If a live check says the repository isn't linked, confirm it appears in your Norma account and that `git remote get-url origin` in your clone points at that same repository.

## Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "norma": {
      "url": "https://api.qualityclouds.ai/mcp"
    }
  }
}
```

Cursor will prompt you to sign in via browser on first connection. As with Claude Code, link the repository in your Norma account first (step 1 of the 5-minute guide), then open your local clone of it.

## Lovable

Lovable supports custom MCP servers as chat connectors, so Norma works with no plugin. In your project, open the connectors panel and add a custom MCP server named `Norma` with server URL `https://api.qualityclouds.ai/mcp` (HTTP transport, OAuth: your browser opens to sign in on first connection). Then send one message, and Lovable keeps it as a standing rule for the project:

> From now on, for every file you create or change, run it through Norma and fix the issues it returns.

One step that is not optional: link the project's GitHub repository in your Norma account, then paste its public GitHub URL in chat so Lovable can tell Norma which repository it is working on. Lovable's own git remote is a tokenised URL it will not send to a third party, so without the public URL live checks do not run and nothing reaches your audit trail. Your code is never stored in your Norma account.

Full guide, including troubleshooting: [docs/LOVABLE.md](docs/LOVABLE.md).

## Links

- Sign up: [norma.qualityclouds.com](https://norma.qualityclouds.com/?utm_source=github&utm_medium=readme&utm_campaign=norma-mcp&utm_content=signup)
- Product: [qualityclouds.ai/norma](https://qualityclouds.ai/norma)
- Community, docs and support: [github.com/qualityclouds/community](https://github.com/qualityclouds/community/discussions)

Built by [Quality Clouds](https://qualityclouds.ai), the AI Code Governance platform, governing 950+ enterprise platform instances since 2017.
