# Personal Server Recon — n8n Instance

**Target:** Friend's self-hosted n8n instance (authorized, written permission)

## What I did
- nmap scan: found SSH (22), HTTP/HTTPS via Caddy (80/443)
- Fingerprinted the app as n8n via JS asset names in page source
- Extracted version (2.35.3) via /metrics endpoint
- Cross-referenced against known n8n CVEs (2026-42226, 2026-42230,
  2026-25049, 2026-21858/"Ni8mare") — all fixed in versions well
  below 2.35.3

## Result
No exploitable code vulnerability — instance was freshly deployed
(2-3 days old) and running a current, patched version. Checked with
the owner about exposed webhooks; none live in current workflows.

## Takeaway
Not every engagement ends in a finding. Confirming a target is
patched and ruling out known CVEs through methodical version-checking
is itself a real, reportable outcome — not a failure. Also a good
lesson in target freshness: newly deployed instances are far less
likely to carry known, unpatched CVEs than long-running ones.
