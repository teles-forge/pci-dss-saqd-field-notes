# PCI DSS SAQ D v4.0.1 - Field Notes

Practical, sanitized field notes from a real PCI DSS SAQ D v4.0.1 engagement on a
high-volume e-commerce payment flow (multiple payment service providers, hundreds of
thousands of monthly transactions, AWS-hosted, JVM back end).

These are working engineering notes, not a compliance product: how to scope, inventory,
manage third parties, justify risk-based frequencies, and - in the most depth - how to
satisfy the two client-side requirements that became mandatory in v4.0.1 and that most teams
underestimate.

## Important

This repository is intentionally vendor-neutral and free of any confidential material.
It contains no secrets, no internal hostnames, no customer or transaction data, no corporate
policy documents, and no penetration-test findings. Everything here is original, reusable
guidance derived from the public PCI DSS v4.0.1 standard plus general implementation
experience, with placeholders ([YOUR-COMPANY], [provider], ...) in place of any specifics.
Adapt to your own environment and validate with your QSA.

## Contents

Foundations
- [`docs/saq-scope-ecommerce.md`](docs/saq-scope-ecommerce.md) - which SAQ applies, and why
  the client-side controls apply even to SAQ A / hosted-field merchants.
- [`docs/scope-and-inventory.md`](docs/scope-and-inventory.md) - defining the CDE, the four
  inventory requirements, data-flow and network diagrams (12.5.1, 12.3.4, 1.2.4, 6.3.3).
- [`docs/service-providers.md`](docs/service-providers.md) - Req 12.8, AOC management, and
  shared responsibility.
- [`docs/targeted-risk-analysis.md`](docs/targeted-risk-analysis.md) - Req 12.3.1, with a
  worked example for payment-page detection frequency.
- [`docs/qsa-prep.md`](docs/qsa-prep.md) - how assessors think and how to prepare evidence.

Client-side payment-page integrity (the deep dives)
- [`docs/req-6.4.3-sri-csp.md`](docs/req-6.4.3-sri-csp.md) - script inventory, authorization,
  Subresource Integrity, and Content Security Policy, enforced at build time.
- [`docs/req-11.6.1-payment-page-tamper-detection.md`](docs/req-11.6.1-payment-page-tamper-detection.md) -
  change-and-tamper detection for the page and its HTTP headers, run on a schedule with
  alerting.
- [`docs/architecture-and-pipeline.md`](docs/architecture-and-pipeline.md) - how the
  build-time controls and the scheduled detector fit together, with diagrams.

Templates
- [`templates/script-inventory.example.json`](templates/script-inventory.example.json)
- [`templates/payment-page-baseline.example.json`](templates/payment-page-baseline.example.json)
- [`templates/targeted-risk-analysis.template.md`](templates/targeted-risk-analysis.template.md)
- [`templates/service-provider-aoc-tracker.template.md`](templates/service-provider-aoc-tracker.template.md)

## Why this matters for v4.0.1

Requirements 6.4.3 (control and assure payment-page scripts) and 11.6.1 (detect tampering of
the page and its headers) did not exist in this form in v3.2.1. v4.0.1 made them mandatory as
of 31 March 2025, and they apply even to merchants who fully outsource the card field, because
the page hosting the iframe or redirect still runs in the consumer's browser and can be
tampered with. The rest of the notes cover the surrounding SAQ D obligations that an assessor
will walk through alongside them.

## Related work upstream

Alongside the compliance work, I contribute to the Jakarta EE / MicroProfile fault-tolerance
ecosystem that these payment services run on, including a merged fix in SmallRye Fault
Tolerance and ongoing work on stereotype/interceptor portability across CDI implementations
(reproducers and proposed TCK/spec portability improvements). See my other repositories.

## Author

Darlysson Teles - Senior Software Engineer, payments and security on the JVM.
LinkedIn: https://www.linkedin.com/in/dev-teles-nascimento/
