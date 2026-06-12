# Service Provider and AOC tracker

> Supports Req 12.8.1, 12.8.4, 12.8.5. One row per provider. Review every 12 months.

| Provider | Service used | In scope because | AOC on file | AOC PCI version | AOC report date | Expires (report date + 12mo) | Responsibility (provider / you / shared) | Owner | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [Cloud IaaS] | [hosting the CDE] | hosts CDE / lower layers | yes/no | v4.0.1 | YYYY-MM-DD | YYYY-MM-DD | shared (see matrix) | [role] | OK / renew |
| [Payment gateway] | [hosted fields] | processes CHD | yes/no | v4.0.1 | YYYY-MM-DD | YYYY-MM-DD | provider | [role] | OK / renew |
| [Monitoring] | [logs/metrics] | holds CDE logs | yes/no | v4.0.1 | YYYY-MM-DD | YYYY-MM-DD | shared | [role] | OK / renew |
| [Pen-test firm] | [testing] | deep access during tests | n/a | n/a | n/a | n/a | provider | [role] | OK |

## Shared-responsibility notes
For each shared row, record which specific requirements the provider owns vs you. For cloud,
the provider typically owns the physical/hypervisor layers; you own configuration, network
rules, encryption settings, identity, and image patching.

## Action checklist
- [ ] Every in-scope provider has a row.
- [ ] Each AOC covers the current PCI version and the services you actually use.
- [ ] No AOC is expired (report date + 12 months).
- [ ] Written agreements acknowledge provider responsibility (12.8.2).
- [ ] Shared-responsibility matrix complete (12.8.5).
