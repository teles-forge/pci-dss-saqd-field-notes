# Requirement 6.4.3 - Script integrity on the payment page

> Manage all payment page scripts that are loaded and executed in the consumer's browser:
> keep an inventory with written justification, authorize each one, and assure their
> integrity. (PCI DSS v4.0.1, 6.4.3)

## The threat it addresses

Client-side payment skimming (Magecart / e-skimming). The attacker does not breach your
backend. They compromise something the browser loads on the payment page - a first-party
bundle, a tag manager, an analytics or chat widget, or a CDN - and add a few lines of
JavaScript that read the card fields and POST them to an attacker-controlled host. The
server logs look perfectly normal. Detection is hard precisely because the malicious code
only runs in the victim's browser.

6.4.3 forces three disciplines: you must know every script that can execute on the payment
page, you must have authorized each one for a reason, and you must be able to prove it has
not been altered between the moment you approved it and the moment it runs.

## The three obligations, made concrete

### 1. Inventory with justification

Produce a living list of every script that can load on the payment page. For each entry
record: source (first party / specific third party), purpose, business justification, and
owner. Generate it from the build, not by hand - a hand-maintained list rots within a
sprint.

A practical source of truth is the bundler output plus an allowlist file checked into the
repo:

```jsonc
// payment-page.scripts.json - reviewed in code review, one entry per allowed script
[
  { "id": "app-bundle",     "origin": "self",                "purpose": "checkout UI",        "owner": "payments-web" },
  { "id": "psp-fields",     "origin": "https://js.psp.example", "purpose": "hosted card fields", "owner": "payments-web" },
  { "id": "error-tracking", "origin": "self-proxied",         "purpose": "JS error capture",   "owner": "platform" }
]
```

Anything observed on the page that is not in this file is, by definition, unauthorized,
and that is exactly what Requirement 11.6.1 (next doc) is built to catch.

### 2. Authorization

Authorization is a human decision recorded in version control. The allowlist above is
changed only through a pull request with a reviewer from the payments team. The PR is the
audit trail your QSA will ask for. Keep third-party scripts to the absolute minimum on the
payment page; every additional origin is additional attack surface you now have to monitor.

### 3. Integrity assurance

Two complementary mechanisms: Subresource Integrity (SRI) for static scripts, and a
Content Security Policy (CSP) to constrain what can load and execute at all.

## Subresource Integrity (SRI)

SRI lets the browser refuse to execute a script whose content does not match a hash you
published. For any static, versioned asset (your own bundles, a pinned third-party file):

```html
<script
  src="https://cdn.example/app.4f9c2a.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
  crossorigin="anonymous"></script>
```

Key points:

- Use sha384 or sha512. The hash is over the exact bytes served.
- `crossorigin` is required for cross-origin scripts so the browser performs a CORS check
  before applying SRI.
- SRI only works on resources whose bytes are stable. It does not work on an endpoint that
  returns different content per request (many PSP loaders do this on purpose). For those,
  rely on CSP allowlisting plus the 11.6.1 detection job rather than SRI.

### Generating SRI at build time

Do not paste hashes by hand. Generate them during the bundle step and inject them into the
HTML, so a content change without a corresponding hash change is impossible. A small
bundler plugin (webpack `SubresourceIntegrityPlugin`, or an equivalent custom step)
computes the hash from the emitted asset and writes the `integrity` attribute. Conceptually:

```js
// pseudo-build-step: compute SRI for every emitted entry script
const crypto = require("crypto");
function sriFor(bytes, algo = "sha384") {
  const digest = crypto.createHash(algo).update(bytes).digest("base64");
  return `${algo}-${digest}`;
}
// emit a manifest the server reads when rendering the payment page
// { "app-bundle": "sha384-...", "checkout": "sha384-..." }
```

The server (or template) then renders each `<script>` with the hash from the manifest. The
hash lives next to the artifact it describes and changes atomically with it.

## Content Security Policy (CSP)

CSP is the enforcement net. On the payment page, lock `script-src` to an explicit allowlist
that matches your inventory, and require integrity where possible:

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://js.psp.example 'nonce-{{REQUEST_NONCE}}';
  style-src 'self';
  connect-src 'self' https://api.psp.example;
  frame-src https://js.psp.example;
  object-src 'none';
  base-uri 'self';
  require-trusted-types-for 'script';
  report-uri /csp-report;
  report-to csp-endpoint;
```

Guidelines that matter in practice:

- Prefer a per-request `nonce` for inline scripts over `unsafe-inline`. Never ship
  `unsafe-inline` on a payment page.
- `connect-src` is as important as `script-src`: it constrains where a script may send
  data, which is the skimmer's exfiltration channel.
- `object-src 'none'` and `base-uri 'self'` close common bypasses.
- Ship CSP in report-only mode first (`Content-Security-Policy-Report-Only`) to collect
  violations without breaking the page, then enforce once the reports are clean.

### Collecting CSP reports

Point `report-uri` / `report-to` at an endpoint you own and feed those reports into your
monitoring. A spike in CSP violations referencing an unknown origin is an early signal of
either a misconfiguration or an active injection attempt. These reports also feed directly
into the 11.6.1 detection story.

## Common pitfalls

- Treating SRI as sufficient. SRI covers static files only; dynamic PSP loaders and
  tag-manager-injected scripts slip past it. CSP plus scheduled detection cover the rest.
- Forgetting the iframe-hosting page. Even with a hosted card field, the parent page can be
  tampered to swap the iframe `src`. The parent page is in scope.
- A CSP so loose it allows `https:` or `unsafe-inline` - that is the same as no CSP for an
  attacker.
- No reporting pipeline, so violations are invisible. Reports are the cheapest detection you
  will ever get; wire them up.
- Hand-maintained hash lists. Automate at build time or it will drift and fail an audit.

## Minimal checklist

- [ ] Build-generated inventory of payment-page scripts, with justification and owner.
- [ ] Allowlist changes gated by code review (the authorization trail).
- [ ] SRI generated at build time for every static script, sha384+.
- [ ] CSP enforced on the payment page: explicit `script-src` and `connect-src`, no
      `unsafe-inline`, `object-src 'none'`, nonce for inline.
- [ ] CSP report endpoint wired into monitoring.
- [ ] CSP rolled out report-only first, then enforced.
