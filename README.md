# Webskan plugin for Claude Code

Audit and re-check the website your agent just shipped. The plugin adds the
[Webskan](https://webskan.com/developers/) remote MCP server (`https://webskan.com/mcp`) and a
`/webskan:site-check` skill that teaches Claude Code the loop: audit, fix the high and medium
findings by id, deploy, re-check, report fixed / still open / new.

What Webskan checks, from the outside, in a real mobile browser: Lighthouse (mobile and desktop),
a crawl of up to 30 pages with SEO checks on each, axe-core accessibility (WCAG 2.1 AA), security
headers and TLS, broken links, forms, the cookie banner, and customer journeys (404 page, menu,
search, cart to checkout). Every finding has a stable id, so the re-check is an exact diff.

## Install

In Claude Code:

```
/plugin marketplace add kamilobstawskiagent-ux/webskan-claude-plugin
/plugin install webskan@webskan
```

Or from the shell:

```
claude plugin marketplace add kamilobstawskiagent-ux/webskan-claude-plugin
claude plugin install webskan@webskan
```

No account and no key are needed for the free plan. If you only want the MCP server without the
skill:

```
claude mcp add --transport http webskan https://webskan.com/mcp
```

## Use

```
/webskan:site-check https://mysite.com
```

or just ask: "audit https://mysite.com and fix the high-severity findings, then recheck".

Tools provided by the server: `scan_website`, `audit_website`, `get_audit`, `recheck_website`.
Details, limits of the free plan and configuration for Cursor, VS Code, Codex CLI and other clients:
https://webskan.com/developers/

## Free plan limits

- `scan_website`: 10 per hour per IP (120 per hour overall); results cached 24 h per host, kept 7 days.
- `audit_website` / `recheck_website`: 2 per IP per 24 h and 1 per host per 24 h; 1-30 pages; results kept 30 days.
- Public websites only - no `localhost`, private networks or sites behind a login.

A paid plan for teams with higher limits (key passed as `Authorization: Bearer <key>`) is coming soon.

## Layout

```
.claude-plugin/marketplace.json      marketplace "webskan"
plugins/webskan/
  .claude-plugin/plugin.json         plugin manifest
  .mcp.json                          remote MCP server
  skills/site-check/SKILL.md         the audit -> fix -> recheck loop
```

## License

MIT. Contact: kontakt@webskan.com
