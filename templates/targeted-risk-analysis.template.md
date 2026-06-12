# Targeted Risk Analysis - [activity name]

> Required by PCI DSS v4.0.1 Req 12.3.1 when a control's frequency is set by the entity.
> One asset/activity per analysis. Review at least every 12 months and on significant change.

- TRA ID: TRA-XXX
- Related requirement: [e.g., 11.6.1 payment page change detection]
- Owner: [role]
- Date: [YYYY-MM-DD]
- Next review: [YYYY-MM-DD]

## 1. Asset(s) being protected
[What is at risk - e.g., the payment page and the HTTP headers received by the browser.]

## 2. Threat(s) addressed
[The threat this control mitigates - e.g., client-side skimming via injected/tampered scripts.]

## 3. Likelihood and impact factors
- [factor, e.g., transaction volume]
- [factor, e.g., number of third-party scripts on the page]
- [factor, e.g., direct cardholder-data exposure if compromised]
- [factor, e.g., page is externally reachable]

## 4. Analysis and chosen frequency
[Reasoning that ties the factors to a frequency. State the residual risk of the default
cadence and why the chosen cadence reduces it acceptably.]

- Chosen frequency: [e.g., daily + on every deploy]

## 5. Review
- Reviewed on: [YYYY-MM-DD] by [role]
- Outcome: [unchanged / adjusted to ...]
