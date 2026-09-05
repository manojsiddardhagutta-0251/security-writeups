# SSRF via Double URL-Encoded Fragment Bypass — Whitelist Filter Circumvention

**Lab:** PortSwigger Web Security Academy — SSRF with whitelist-based input filter (Expert)
**Vulnerability class:** Server-Side Request Forgery (SSRF)

## Summary
The application validates a `stockApi` parameter against a whitelist of allowed hostnames. By exploiting a parsing inconsistency between the validation logic and the actual request-execution logic, it was possible to bypass the whitelist and reach an internal admin interface at `localhost`.

## Vulnerability Context
- The app's stock-check feature fetches a URL supplied via the `stockApi` parameter.
- Only the hostname `stock.weliketoshop.net` is permitted by the whitelist filter.
- Goal: reach `http://localhost/admin/delete?username=carlos`, which is restricted to internal, unauthenticated (localhost-only) access.

## Attempted Approaches

**1. Plain `@` credential injection**

    stockApi=http://stock.weliketoshop.net@localhost/admin

Result: `400 Bad Request`. The validator correctly parses everything after `@` as the real host and rejects `localhost` as non-whitelisted.

**2. Raw `#` fragment injection**

    stockApi=http://stock.weliketoshop.net#@localhost/admin

Result: `500 Internal Server Error`. The raw `#` character is flagged as malformed input before reaching the fetch logic.

**3. Subdomain prefixing**

    stockApi=http://stock.weliketoshop.net.localhost/admin

Result: `400 Bad Request`. The validator correctly parses the actual base domain rather than relying on a naive "starts with" string check.

## Working Exploit

    stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos

Result: `200 OK` — request reached the internal admin interface and successfully deleted user `carlos`.

## Root Cause

The application has two separate code paths handling this URL:

- **Validation logic** decodes the URL string once. It sees `%2523` as a harmless literal string within the "credentials" portion of the URL, and correctly identifies `stock.weliketoshop.net` (the part after `@`) as the host — which passes the whitelist check.
- **Request-execution logic** decodes the URL twice. On the second decode, `%2523` → `%23` → `#`, introducing a real fragment marker. This causes the URL to be reinterpreted: everything from the fragment onward (`@stock.weliketoshop.net/admin/...`) is treated as discarded fragment content, and the actual connection target becomes `localhost:80`. The path (`/admin/delete?username=carlos`) is still applied to the final request due to how the fetching library reconstructs the request from the malformed string.

This decode-count mismatch between validation and execution is the core vulnerability — a classic parser differential.

## Impact
Full SSRF bypass despite an active whitelist defense, enabling access to internal-only admin functionality and unauthorized state-changing actions (user deletion demonstrated).

## Remediation
- Parse URLs using a single, well-tested URL parsing library — do not build custom string-matching validation logic.
- Ensure validation and execution logic decode input the exact same number of times, ideally by validating the *already-fully-decoded* URL that will actually be used for the request, not a separately-decoded copy.
- Avoid whitelist logic based on `startswith`/`contains` string checks; instead extract and compare the actual parsed hostname component.
