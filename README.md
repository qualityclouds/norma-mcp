<!-- mcp-name: ai.qualityclouds/norma -->
# Norma MCP Server

AI code governance rules injected into Cursor, Claude Code and other MCP clients at generation time.

Norma connects your AI development tool to a governance rule library built around the error patterns AI-generated code typically contains. When the model generates code, the rules for your detected stack are already in its context, so the output follows them from the first line.

- Remote server (Streamable HTTP): `https://api.qualityclouds.ai/mcp`
- Auth: OAuth. Sign in via browser on first connection. Permanent free tier.
- Works with Cursor, Claude Code, Lovable, Replit and any MCP-compatible client
- Stacks covered: TypeScript, Python, Java, Node, React, Supabase and more

## What it does

Rulesets are stack-specific and activate automatically for the stack the server detects. A single repository scan activates rulesets across six functional areas: Security, Performance, Scalability, Manageability, Maintainability and Architecture. The Supabase ruleset alone covers 12 rules, including no service-role keys outside the server, no hardcoded keys or project URLs, mandatory error checks on every mutation, no client-side JWT decoding, and rate limiting on edge functions.

The server exposes six tools:

| Tool | What it does |
|------|--------------|
| `link_repository` | Links the current workspace to Norma. Called once, on first connection in a workspace, before any other tool |
| `get_rulesets` | Detects your repository's tech stack, with no configuration, and returns the applicable rulesets, one per language or framework |
| `get_rules_for_ruleset` | Returns the rules in one ruleset: each rule's ID, name, severity, impact area, description and fix guidance |
| `live_check` | Deterministic check of one file against your organization's rules. Same file, same rules, same verdict every time. Nothing is stored |
| `get_open_issues` | Returns the open issues from the last full scan of the linked repository, each with the context needed to fix it |
| `register_applied_actions` | Records what was done in your compliance audit trail: rules verified compliant, violations fixed (file and lines), violations prevented during generation, and which model did the work |

Every session produces a structured record of what was checked, fixed and prevented. Your repository's Production-Ready Score and full findings live in your workspace at [norma.qualityclouds.com](https://norma.qualityclouds.com).

## How it works

A coding agent connected to Norma follows this sequence:

1. `link_repository`: first connection in each workspace, before any other call.
2. `get_rulesets`: at the start of any coding task.
3. `get_rules_for_ruleset`: for each relevant ruleset ID; never skipped.
4. The agent writes or modifies code, with those rules in context.
5. `live_check`: after each file is created or modified, before moving on.
6. `register_applied_actions`: after the task, using the exact rule IDs from step 3.

To work through standing issues instead of writing new code, the agent calls `get_open_issues` and fixes the violations from the repository's last full scan.

## Try it in 5 minutes (Claude Code)

1. Add the server:

```bash
claude mcp add --scope user --transport http norma https://api.qualityclouds.ai/mcp
```

2. Check it connected:

```bash
claude mcp list
```

3. On first use, your browser opens to sign in. Free to start, no credit card.
4. Open a session in any repo and ask Claude Code to review a file against your coding standards. It will link the repository, fetch the rules for your stack, and register the results back to your workspace.

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

Cursor will prompt you to sign in via browser on first connection.

## Links

- Sign up: [norma.qualityclouds.com](https://norma.qualityclouds.com)
- Product: [qualityclouds.ai/norma](https://qualityclouds.ai/norma)
- Community, docs and support: [github.com/qualityclouds/community](https://github.com/qualityclouds/community/discussions)

Built by [Quality Clouds](https://qualityclouds.ai), the AI Code Governance platform, governing 950+ enterprise platform instances since 2017.
