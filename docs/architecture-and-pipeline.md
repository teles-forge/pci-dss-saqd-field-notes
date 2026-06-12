# Architecture and pipeline

How the build-time integrity controls (6.4.3) and the scheduled detection mechanism
(11.6.1) fit together. Diagrams are vendor-neutral; the notes in brackets show the
tooling used in the original engagement.

## Build time: producing an integrity-assured payment page (6.4.3)

```mermaid
flowchart LR
    A[Source: payment-web<br/>+ scripts allowlist] --> B[Bundler build]
    B --> C[Emit hashed assets<br/>app.4f9c2a.js]
    C --> D[Compute SRI per asset<br/>sha384/512]
    D --> E[Write SRI manifest]
    E --> F[Server renders payment page<br/>script integrity + nonce + CSP]
    A --> G[scripts allowlist<br/>reviewed in PR]
    G -.authorization trail.-> F
    F --> H[Deploy<br/>ECS Fargate / CDN]
```

The allowlist (authorization) and the SRI manifest (integrity) are both build artifacts
checked or generated in CI, so the rendered page cannot reference a script that was not
authorized and hashed.

## Run time: detecting drift on the live page (11.6.1)

```mermaid
flowchart TD
    S[Scheduler<br/>daily + post-deploy] --> D[Detector fetches LIVE payment URL<br/>external vantage point]
    D --> E[Extract headers + executed scripts<br/>+ connect/iframe origins]
    E --> N[Normalize volatile fields<br/>nonce/token/price -> *]
    N --> C{Diff vs baseline}
    C -- match --> OK[Emit pass metric]
    C -- drift --> AL[Emit fail metric + event with diff]
    AL --> M[Monitor / alert]
    M --> P[Page on-call owner<br/>severity-graded]
    B[baseline.payment-page.json<br/>derived from 6.4.3 inventory + CSP] --> C
    PR[Reviewed PR updates baseline<br/>on intended change] --> B
```

[Scheduler: Azure DevOps pipeline + scheduled function. Metrics, events and monitors:
Datadog. Hosting and edge: AWS ECS Fargate, WAF, CloudFront-style CDN, CloudTrail for
audit. AppConfig for runtime flags.]

## How the two halves reinforce each other

```mermaid
flowchart LR
    subgraph Intent[6.4.3 - intent, build time]
      INV[Authorized inventory]
      SRI[SRI hashes]
      CSP[CSP policy]
    end
    subgraph Verify[11.6.1 - verification, run time]
      BL[Baseline]
      DET[Scheduled detector]
    end
    INV --> BL
    CSP --> BL
    SRI --> BL
    DET -->|drift| ALERT[Alert + runbook]
    DET -->|clean| EVID[Weekly evidence for QSA]
```

The detector's clean runs double as the periodic evidence a QSA expects for 11.6.1, and the
baseline is generated from the same inventory and CSP that satisfy 6.4.3 - one source of
truth, two requirements covered.

## Design principles that kept it maintainable

- One repository holds the allowlist, the CSP, and the detection baseline. A change to any
  of them is one reviewed PR, which is also the authorization record.
- Everything that can be generated is generated (SRI hashes, inventory). Nothing
  security-relevant is hand-edited on a payment page.
- The detector observes the page from outside, the way a customer does, not from inside the
  build. Injection happens at the edge and in third-party tags, which an internal-only check
  never sees.
- Alerts carry the diff and are graded by severity, so responders act on signal, not noise.
- Report-only first, then enforce (for CSP); baseline-then-alert (for detection). Roll out
  controls without breaking checkout.
