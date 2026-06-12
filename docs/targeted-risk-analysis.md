# Targeted Risk Analysis (Requirement 12.3.1)

v4.0.1 lets you set the frequency of several periodic activities yourself, instead of
following a fixed number, as long as you justify the chosen frequency with a documented
Targeted Risk Analysis (TRA). If you use a customised frequency anywhere, you must have the
matching TRA, and assessors will ask for it.

## Where a TRA is required

Any control whose text says the frequency is defined "in the entity's targeted risk
analysis", including:

- payment page change/tamper detection frequency (11.6.1);
- log review frequency for lower-risk components (10.4.2);
- anti-malware scan frequency (5.2.3.1 / 5.3.2.1);
- frequency for addressing lower-severity vulnerabilities (6.3.3 / 11.3.1.1);
- application and system account password change frequency where applicable (8.6.3);
- POI/payment-terminal and other periodic checks defined by TRA.

If you simply adopt the standard's default cadence (for example, weekly for 11.6.1), you do
not need a TRA for it. The TRA is only to justify a different, risk-based frequency.

## What a TRA must contain (12.3.1)

For each asset/activity covered, document:

1. The asset(s) being protected.
2. The threat(s) the control addresses.
3. The factors that contribute to likelihood and/or impact.
4. The resulting analysis and the frequency you chose.
5. A review at least every 12 months and on significant change.

## Worked example: payment page change-detection frequency (11.6.1)

- Asset: the payment page and the HTTP headers the browser receives.
- Threat: client-side skimming via injected or tampered scripts (Magecart), which can
  exfiltrate cardholder data within hours of injection.
- Likelihood/impact factors: high transaction volume, multiple third-party scripts, an
  externally reachable page, direct cardholder-data exposure if compromised.
- Analysis: weekly detection could leave a skimmer live for up to seven days. Given the
  volume and direct CHD exposure, the residual risk of a weekly cadence is not acceptable.
- Chosen frequency: daily automated detection, plus a run on every deploy.
- Review: at least every 12 months and whenever the page's third-party footprint changes.

That single page is the kind of artifact an assessor expects when you run detection daily
rather than weekly. See `templates/targeted-risk-analysis.template.md`.

## Practical notes

- Keep each TRA short and specific; one asset/activity per analysis is easier to maintain
  and to defend than a single sprawling document.
- Tie the chosen frequency to something observable (volume, exposure, number of third-party
  scripts) so the justification still holds when someone re-reads it in a year.
- Store TRAs in version control so the annual review is a visible, dated change.
