# Requirement 11.6.1 - Payment page change and tamper detection

> Deploy a change-and-tamper-detection mechanism that alerts personnel to unauthorized
> modification of the HTTP headers and the contents of the payment page as received by the
> consumer browser. Evaluate at least once every seven days (or at a defined,
> risk-based frequency). (PCI DSS v4.0.1, 11.6.1)

Where 6.4.3 is about controlling what you intend to load, 11.6.1 is about detecting when
reality diverges from that intent - including changes you did not make.

## What the standard actually asks for

Three properties:

1. You look at the payment page the way the consumer's browser sees it: the rendered HTTP
   response headers and the script content that executes.
2. You compare it against a known-good baseline.
3. On an unauthorized difference, a human is alerted. Cadence is weekly at minimum; daily
   is the defensible default for a high-volume payment flow.

It is not enough to scan your repo or your build output. The check must observe what is
actually delivered to the browser, after CDN, edge, WAF, and any tag manager have had their
say. That is where injection happens.

## What to monitor

Build the baseline from the two things an e-skimmer must touch:

### HTTP response headers (as received by the browser)

- `Content-Security-Policy` - the single most security-relevant header. Any weakening
  (an added origin, `unsafe-inline` creeping in, a removed directive) is high severity.
- `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy`,
  `X-Frame-Options` / `frame-ancestors`.
- `Set-Cookie` flags on session cookies (`Secure`, `HttpOnly`, `SameSite`).
- Caching headers on the HTML document (the payment page should not be cached).

### Page content that executes

- The set of `<script>` elements: their `src` origins and `integrity` attributes.
- The set of external origins the page connects to (compare against the 6.4.3 inventory).
- Inline script hashes / nonce usage.
- The payment iframe `src` origin, if a hosted field is used (a classic skimming move is to
  repoint the iframe to a look-alike origin).

## Baseline and diff approach

Capture a structured fingerprint, not a raw byte diff (the HTML legitimately varies per
request - nonces, CSRF tokens, prices). Normalize, then compare the security-relevant
subset.

```jsonc
// baseline.payment-page.json - committed, changed only via reviewed PR
{
  "headers": {
    "content-security-policy": "default-src 'self'; script-src 'self' https://js.psp.example 'nonce-*'; connect-src 'self' https://api.psp.example; object-src 'none'",
    "strict-transport-security": "max-age=63072000; includeSubDomains; preload",
    "x-content-type-options": "nosniff"
  },
  "scripts": [
    { "origin": "self",                  "integrity": "required" },
    { "origin": "https://js.psp.example", "integrity": "n/a-dynamic" }
  ],
  "connectOrigins": ["self", "https://api.psp.example"],
  "iframeOrigins": ["https://js.psp.example"]
}
```

The detector fetches the live page (headless browser or a fetch that records response
headers), extracts the same structure, normalizes volatile parts (nonces become `*`,
prices and tokens are stripped), and diffs against the baseline. Any of these is an alert:

- a script origin not in the baseline / not in the 6.4.3 inventory;
- a missing or altered `integrity` on a script that should have one;
- a CSP that is weaker than baseline (new origin, removed directive, `unsafe-inline`);
- a new `connect-src` or iframe origin;
- a security header that disappeared or weakened.

## Run it like a pipeline

A defensible setup runs the detector on a schedule and on every deploy:

1. Scheduled job (daily, or more often) executes the detector against the live payment URL
   from an external vantage point, so it sees the same edge/CDN path a real user does.
2. The detector emits a pass/fail plus a structured diff.
3. Results are shipped to your observability platform as a metric and an event. A failed
   comparison opens an alert that pages the on-call owner.
4. The same detector runs as a post-deploy gate, so an intended change updates the baseline
   through a reviewed PR rather than silently triggering a 2 a.m. page.

Mapping to the original engagement's tooling: the scheduled detector ran in CI (Azure
DevOps) and as a small scheduled function; results and alerts lived in Datadog (a monitor
on the comparison metric, plus events carrying the diff). Any equivalent scheduler +
alerting stack works - GitHub Actions + PagerDuty, cron + Opsgenie, and so on.

## Alerting and response

- Severity by signal: CSP weakening and unknown script/connect origins are high severity
  (possible active skimming) and should page immediately. A changed cache header is lower.
- The alert payload carries the diff so the responder sees exactly what changed without
  re-running anything.
- Response runbook: confirm whether the change was an authorized deploy (check the PR that
  should have updated the baseline). If not, treat as a potential compromise - rotate, pull
  the page or roll back, preserve evidence, and follow the incident process.
- Tune to near-zero false positives. A noisy detector gets muted, and a muted detector
  detects nothing. Normalizing volatile fields is what keeps it quiet.

## Relationship to 6.4.3

They are two halves of one control:

- 6.4.3 defines intent: the authorized inventory, SRI, and CSP.
- 11.6.1 verifies reality against that intent on a schedule and alerts on drift.

The baseline for 11.6.1 is literally derived from the 6.4.3 inventory and CSP. Keep them in
the same repository so a change to one forces a reviewed update to the other.

## Minimal checklist

- [ ] Detector observes the live payment page as the browser receives it (headers + executed
      scripts), from an external vantage point.
- [ ] Structured, normalized baseline committed and changed only via reviewed PR.
- [ ] Scheduled at least weekly (daily recommended) and run as a post-deploy gate.
- [ ] Alerts page a named owner, carry the diff, and are graded by severity.
- [ ] Documented response runbook tied to the incident process.
- [ ] False-positive rate driven to near zero by normalizing volatile fields.
