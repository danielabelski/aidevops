# Bot Management Gotchas

## Bot Score = 0

- Means Bot Management did not run, not that the request scored 100.
- Common causes: internal Cloudflare request, Worker-routed Orange-to-Orange traffic, or request handling that finishes before Bot Management runs.
- Fix: trace the request path and ensure Bot Management executes in the request lifecycle.

## JavaScript Detections Not Working

If `js_detection.passed` is always `false` or `undefined`, check:

- CSP does not allow `/cdn-cgi/challenge-platform/`
- First page visit; JSD needs an HTML page first
- JavaScript disabled or blocked by an ad blocker
- JSD not enabled in the dashboard
- Rule action is `Block`; JSD requires `Managed Challenge`

**CSP fix:**

```txt
Content-Security-Policy: script-src 'self' /cdn-cgi/challenge-platform/;
```

## False Positives

1. Check Bot Analytics for affected IPs and paths.
2. Identify the detection source (ML, heuristics, and so on).
3. Add an exception rule when the problem is isolated:

```txt
(cf.bot_management.score lt 30 and http.request.uri.path eq "/problematic-path")
Action: Skip (Bot Management)
```

4. Allowlist by IP, ASN, or country if needed.

## False Negatives

1. Raise the enforcement threshold (for example 30 → 50).
2. Enable JavaScript Detections.
3. Add JA3/JA4 fingerprinting rules.
4. Use rate limiting as a fallback control.

## Verified Bot Blocked

- Usually caused by WAF Managed Rules, not Bot Management itself.
- Yandex bot verification can fail during Cloudflare IP updates for up to 48 hours.
- Fix: create a WAF exception for the specific rule ID, then verify the bot with reverse DNS.

## JA3/JA4 Missing

- Confirm the request is HTTPS/TLS traffic.
- Check for Worker-routed or Orange-to-Orange traffic.
- Confirm Bot Management actually ran.
- JA3/JA4 only exists for TLS traffic that reaches Bot Management.

## Detection Limits

### Bot score

- `0` means not computed.
- First requests may not include JSD data.
- Scores are probabilistic; false positives and false negatives still happen.

### JavaScript Detections

- Does not work on the first HTML page visit.
- Requires a JavaScript-enabled browser.
- Strips ETags from HTML responses.
- Can break with restrictive CSP.
- Does not support `<meta>` CSP tags.
- Does not support WebSocket endpoints.
- Native mobile apps will not pass JSD.

### JA3/JA4 fingerprints

- HTTPS/TLS only.
- Missing on Worker-routed traffic.
- Not unique per user; clients can share fingerprints.
- Can change when browsers or libraries update.

## Plan Restrictions

| Feature | Free | Pro/Business | Enterprise |
|---------|------|--------------|------------|
| Granular scores (1-99) | No | No | Yes |
| JA3/JA4 | No | No | Yes |
| Anomaly Detection | No | No | Yes |
| Corporate Proxy detection | No | No | Yes |
| Verified bot categories | Limited | Limited | Full |
| Custom WAF rules | 5 | 20/100 | 1,000+ |

## Technical Constraints

- Max 25 WAF custom rules on Free, though exact limits vary by plan.
- Workers CPU time limits still apply to bot-related logic.
- Bot Analytics is sampled at 1-10%.
- Maximum analytics history is 30 days.
- JSD still depends on CSP allowing `/cdn-cgi/challenge-platform/`.
