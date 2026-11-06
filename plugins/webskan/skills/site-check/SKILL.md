---
name: site-check
description: Check a live, publicly reachable website with the Webskan MCP tools (scan_website, audit_website, get_audit, recheck_website) after a deploy - run a full audit, fix the high and medium findings by id, re-check, and report fixed / still open / new. Use when the user asks to audit, check, test or verify a deployed site, its SEO, speed, accessibility, security headers or broken links.
---

# Webskan site check

Webskan is a remote MCP server (`https://webskan.com/mcp`) that looks at a deployed website from the
outside, in a real mobile browser: Lighthouse (mobile and desktop), a crawl of up to 30 pages with SEO
checks on each, axe-core accessibility, security headers and TLS, broken links, forms, the cookie
banner and customer journeys (404 page, menu, search, cart to checkout). Every finding has a stable id,
so a re-check can say exactly what was fixed.

## Tools

| Tool | Input | What you get |
|---|---|---|
| `scan_website` | `url` | Mobile Lighthouse scores, key metrics, basic issues, `resultUrl`. About 30 s. Cached 24 h per host. May return `status: "pending"` + `scanId` - call again in 30 s. |
| `audit_website` | `url`, optional `pages` (1-30, default 15) | `auditId`, `status: "queued"`, `etaSeconds`, `pollWith: "get_audit"`. Asynchronous. |
| `get_audit` | `auditId` | Status while running; when done: `summary`, `lighthouse`, `findings` (high to low, each with `id`, `area`, `severity`, `title`, `detail`, `pages`), `pagesCrawled`, `finalUrl`. For a re-check also `diff`: `fixed`, `open`, `new`, `unverified`, `lighthouseBefore`/`lighthouseAfter`. |
| `recheck_website` | `auditId` | A new audit of the same URL and page count linked to the parent. Returns like `audit_website`. |

## The loop

1. **Confirm the target.** The URL must be public (https, reachable from the internet). Never pass
   `localhost`, `127.0.0.1`, private IPs, `*.local`, tunnels that need a header, or staging behind a
   login - the server rejects them with `isError: true`. If the user only has a local build, deploy
   a preview first.
2. **Quick look (optional).** `scan_website` when the user wants a 30-second answer (scores, biggest
   issue). For anything to fix, go straight to the audit.
3. **Audit.** `audit_website({ url, pages })`. Use `pages: 5-10` for a landing page, `15` (default)
   for a typical business site, up to `30` for a shop. Note the `auditId`.
4. **Wait for the result.** Call `get_audit({ auditId })` every 30-60 s until `status` is `done`
   (use `etaSeconds` as a hint; a typical audit takes a few minutes, slow sites longer). Do not start a
   second audit of the same host while one is running - the free plan allows 1 audit per host per
   24 h and a second call returns the same `auditId` anyway. If `status` is `error`, read the message:
   the site blocked the crawler, timed out (15 min) or was unreachable.
5. **Fix by id.** Work through `high` findings first, then `medium`. Each finding has the page(s) it
   occurs on and a `detail` with what to change; accessibility findings include the selector and the
   HTML of the failing element. Map each id to the file or template in the repo, change it, keep a
   list `id -> what you changed`. Skip `low`/`info` unless the user asks. Do not "fix" Lighthouse
   metric findings (`perf.*.mobile`) by chasing a score - fix the concrete causes (image size,
   render-blocking CSS, server response time) listed in the details.
6. **Deploy, then re-check.** After the changes are live, call `recheck_website({ auditId })` with the
   id of the first audit, then poll `get_audit` with the new id. Read `diff.fixed`, `diff.open`,
   `diff.new` and `diff.unverified`.
7. **Report.** Give the user: fixed (with what you changed), still open (and why - needs content,
   needs a server setting, out of scope), new (usually side effects of the fixes), and Lighthouse
   before -> after. Include the `resultUrl`/`auditId` so they can look it up again. Say which findings
   you deliberately left.

## Rules

- Public URLs only; never scan localhost or private networks.
- Always wait for `get_audit` to report `done` before reading findings; do not guess from a partial result.
- Lighthouse scores vary by a few points between runs. Treat differences of up to about 5 points as
  noise and never report them as a fix or a regression; use the concrete metrics and findings instead.
- Findings from a page that did not respond during the re-check land in `unverified` - say so, do not
  count them as fixed.
- Respect the free plan: 10 scans per hour per IP, 2 audits per IP per 24 h, 1 audit per host per
  24 h (re-checks count as audits). If a limit error comes back, tell the user when to retry rather
  than retrying in a loop.
- Results are kept 7 days (scan) and 30 days (audit). Save the `auditId` in the hand-off note if the
  user will re-check later.
- The server only measures the public site. It never sees the repository, so every code change is
  yours to make and explain.

More: https://webskan.com/developers/
