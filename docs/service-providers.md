# Service providers (Requirement 12.8)

Almost no one runs a payment flow alone. Cloud hosting, payment gateways, CDN/WAF,
monitoring, identity, EDR, and pen-test firms are all third parties, and Requirement 12.8
makes managing them your responsibility.

## What counts as a service provider

Any entity that stores, processes, or transmits cardholder data on your behalf, OR that
could affect the security of the CDE. The "could affect security" clause is deliberately
broad and catches more than the payment gateways:

| Provider type | Why in scope | Examples (generic) |
| --- | --- | --- |
| Cloud infrastructure | Hosts the CDE, manages the lower layers | IaaS/PaaS provider |
| Payment gateways | Receive and process actual cardholder data | Hosted-fields / redirect PSPs |
| Network / identity / VPN | Manage remote access and MFA into the CDE | Parent-company IdP, VPN |
| Monitoring / observability | Hold CDE logs and metrics; possible exfiltration vector | APM / logging platform |
| Security testing | Has deep access during testing | Pen-test firm / QSA |
| EDR / endpoint | Runs on production hosts and laptops | Endpoint security vendor |
| CDN / WAF (external) | Sits in front of payment pages | Edge/WAF provider |

## The obligations

| Requirement | Obligation |
| --- | --- |
| 12.8.1 | Maintain a list of service providers, with a description of the service provided |
| 12.8.2 | Written agreements that include the providers' acknowledgement of their responsibility for the CHD they handle or could affect |
| 12.8.3 | A due-diligence process before engaging a provider |
| 12.8.4 | Monitor each provider's PCI DSS compliance status at least every 12 months |
| 12.8.5 | Maintain which PCI DSS requirements are managed by each provider, by you, or shared |

## The AOC, in practice

The Attestation of Compliance (AOC) is a signed document, produced from a provider's own
PCI DSS assessment, that proves they passed and states what scope it covered. It is your
primary evidence under 12.8.4. When you collect one, check:

1. It covers the current PCI DSS version you are assessed against.
2. It is not expired. An AOC is valid for twelve months from the date its report was written
   (look for the "report was written on" date, not the signature date).
3. Its scope includes the specific services you actually use.
4. It is retained on file (upload it to your assessment portal).

For large cloud providers, the AOC is usually available through their compliance portal or
artifact service rather than by emailing an account manager. Budget time to renew expired
gateway AOCs early; chasing them at audit time is a classic source of delay.

## Shared responsibility

12.8.5 is where most of the real work is: for each provider, write down which requirements
they own, which you own, and which are shared. For cloud especially, the provider owns the
physical and hypervisor layers while you own everything you configure on top (network rules,
encryption settings, IAM, patching of your images). Capture this in a matrix so that at
audit you can point to exactly who is responsible for each control.

See `templates/service-provider-aoc-tracker.template.md` for a tracker structure.
