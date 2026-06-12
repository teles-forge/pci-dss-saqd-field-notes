# Working with a QSA: preparation notes

Generic, vendor-neutral notes on preparing for assessment sessions with a Qualified
Security Assessor (QSA: the independent, PCI-certified auditor who validates your
compliance). No assessor, firm, or client specifics here, just patterns that hold across
engagements.

## How assessors tend to think

- They validate evidence, not assertions. "We do X" is worth little without an artifact
  that shows X happening on a date.
- They follow the data. Expect them to start from the data-flow diagram and pull on threads:
  if cardholder data crosses a boundary, they will ask how that boundary is controlled and
  monitored.
- They accept reasonable, well-documented decisions. A customised frequency with a TRA, or a
  control implemented differently but meeting intent, is fine if you can explain it.
- They dislike surprises and inconsistency. The fastest way to lose time is to have the
  inventory, the network diagram, and the live environment disagree.

## What to have ready before each session

- The data-flow diagram and current network diagram, consistent with the live environment.
- The component inventory (12.5.1) and software list, generated from source of truth.
- The service-provider list with current AOCs and a shared-responsibility matrix (12.8).
- Evidence for periodic activities with dates: log reviews, scans, access reviews, payment
  page detection results.
- Any TRAs that justify customised frequencies (see `docs/targeted-risk-analysis.md`).
- For client-side controls: the script inventory, the enforced CSP, SRI manifest, and recent
  detection-job results (6.4.3 and 11.6.1).

## Common questions by area (be ready to show evidence, not just answer)

- Scope: "Walk me through how cardholder data flows." "What is in the CDE and how is the rest
  segmented?"
- Inventory: "Show me the component inventory. How is it kept current?"
- Service providers: "Show the AOC for this provider. Does its scope cover the service you
  use? Is it current?"
- Client-side (e-commerce): "What scripts run on the payment page and why? How do you assure
  their integrity? How would you know if the page or its headers were tampered with?"
- Logging/monitoring: "Show a recent log review. Who does it and how often? Why that
  frequency?"
- Vulnerability management: "Show your last scans and how findings were remediated within the
  required timeframes."
- Access: "Show the last access review. How is least privilege enforced and MFA applied to
  the CDE?"

## Session hygiene that saves time

- Bring one owner per topic who can both answer and show the artifact live.
- Pre-stage evidence in the assessment portal so sessions are a walkthrough, not a scavenger
  hunt.
- When something is not yet done, say so and show the plan and date. Assessors handle honest
  gaps far better than discovering them.
- Capture every action item with an owner during the session; reconcile them the same day.
