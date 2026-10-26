# galicrisp.pl audit status

**Target:** https://www.galicrisp.pl/
**Attempted:** 2026-10-26 (UTC)
**Result:** NOT RUN. The target host is unreachable from the audit environment.

No findings, scores, or recommendations have been produced. Nothing in this
folder describes the site itself; it only records the attempt and the runbook.

## Evidence

| Check | Outcome |
|---|---|
| `curl -L https://www.galicrisp.pl/` | `CONNECT tunnel failed, response 403` (egress proxy policy denial) |
| `curl -L https://galicrisp.pl/` | `CONNECT tunnel failed, response 403` |
| Proxy relay log | `www.galicrisp.pl:443` and `galicrisp.pl:443` both logged as `gateway answered 403 to CONNECT (policy denial or upstream failure)` |
| WebFetch tool | `getaddrinfo ENOTFOUND www.galicrisp.pl` |
| Web search `site:galicrisp.pl` | No indexed pages returned |

The container has no direct DNS (every hostname fails to resolve locally), so
all outbound traffic depends on the proxy, which denied this host.

## Unblock

1. In the cloud environment settings (session title bar, environment menu, Edit),
   under **Network access** add `www.galicrisp.pl` and `galicrisp.pl` to
   Allowed domains, or choose a broader access level. Keep the package
   managers box ticked. Docs: https://code.claude.com/docs/en/cloud-environments#network-access
2. Confirm the domain is live. Search engines currently return no pages for
   galicrisp.pl, which is consistent with a new or unlaunched site. The
   bakery's existing product content is on galicia.com.pl.

## Runbook (once reachable)

Run from the repository root with `CLAUDE_PLUGIN_ROOT` set to the repo path.

```bash
pip install -r requirements.txt            # playwright, matplotlib, weasyprint
export CLAUDE_PLUGIN_ROOT="$PWD"
export CLAUDE_PLUGIN_DATA="$HOME/.local/share/claude-seo"
OUT=galicrisp.pl-audit
URL=https://www.galicrisp.pl/

"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run render_page.py "$URL" --mode auto --json   > "$OUT/findings/render.json"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run fetch_page.py "$URL" --output "$OUT/findings/home.html"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run parse_html.py "$OUT/findings/home.html" --url "$URL" --json > "$OUT/findings/parse.json"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run sitemap_discovery.py "$URL" --json         > "$OUT/findings/sitemap.json"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run agentic_check.py "$URL" --json             > "$OUT/findings/agentic.json"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run content_quality.py "$OUT/findings/home.html" --json > "$OUT/findings/content.json"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run preload_check.py "$URL" --json             > "$OUT/findings/preload.json"
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run capture_screenshot.py "$URL" --all --output "$OUT/screenshots"
# Optional, needs a PageSpeed API key: pagespeed_check.py "$URL" --json
```

Then run `/seo audit https://www.galicrisp.pl/` so the orchestrator fills
`audit-data.json`, `FULL-AUDIT-REPORT.md`, and `ACTION-PLAN.md`, and generate
the PDF with:

```bash
"$CLAUDE_PLUGIN_ROOT/scripts/claude-seo" run google_report.py --type full \
  --data galicrisp.pl-audit/audit-data.json --domain galicrisp.pl \
  --output-dir galicrisp.pl-audit/
```
