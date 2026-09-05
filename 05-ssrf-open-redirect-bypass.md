# SSRF via Open Redirect Chaining — Whitelist Filter Bypass

**Lab:** PortSwigger Web Security Academy — SSRF with filter bypass via open redirection (Practitioner)
**Vulnerability class:** Server-Side Request Forgery (SSRF)

## Summary
The application whitelists the `stockApi` parameter against a trusted domain. That same trusted domain contains an unrelated open redirect vulnerability, which was chained with the SSRF to reach an internal target.

## Exploit

    stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://localhost/admin

## How it works
1. The filter validates the URL and confirms the domain (`weliketoshop.net`) is whitelisted.
2. The server fetches the URL, which triggers an existing open redirect on that domain, redirecting to `http://localhost/admin`.
3. The server's HTTP client follows the redirect automatically, resulting in the real request landing on the internal target.

## Root Cause
Whitelist validation only checks the initially submitted URL, not the final destination after any redirects are followed. Any open redirect on a whitelisted domain effectively nullifies the whitelist.

## Remediation
- Disable automatic redirect-following in server-side HTTP fetch logic, or re-validate the destination URL after every redirect hop.
- Fix the underlying open redirect vulnerability independently, as it is a separate flaw with its own risk.
