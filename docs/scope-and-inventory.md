# Scope and inventory

Before any control can be assessed, you must be able to prove you know what is in the
Cardholder Data Environment (CDE: every system that stores, processes, or transmits
cardholder data, or that could affect its security). PCI DSS has four inventory-style
requirements that all feed this goal. Get them right early; everything else depends on them.

## The four inventories

| Requirement | What it asks | Typical cadence |
| --- | --- | --- |
| 12.5.1 | Inventory of all in-scope system components, with type and PCI DSS function | Ongoing; confirmed at scope review |
| 12.3.4 | Review of hardware and software to confirm PCI DSS still applies / is current | Every 12 months |
| 1.2.4 | Accurate network diagram(s) showing all connections to and from the CDE | 12 months (6 for service providers) |
| 6.3.3 | All software kept protected against known vulnerabilities (implies a software list) | Ongoing; check for patches monthly |

## Defining scope

Scope is the set of components in the CDE plus anything connected to or that can affect it.
Work it out from the data flow, not from the org chart.

1. Map every path cardholder data takes, from capture in the browser to the payment service
   provider. Draw it. This is your data-flow diagram.
2. Everything that touches that path is in the CDE. Everything that can reach or influence
   the CDE (admin jump hosts, identity, monitoring, CI/CD that deploys CDE code) is
   connected-to or security-impacting, and therefore in scope.
3. Everything else is out of scope only if it is genuinely segmented from the CDE. If there
   is no enforced segmentation, assume it is in scope.

> A common and defensible posture for a fully cloud-hosted environment: the CDE is the set
> of services on the payment path plus their data stores, network controls, secrets, and the
> pipelines that deploy them. Segmentation is enforced at the network layer (separate
> networks/subnets, security groups, WAF) and proven with the network diagram.

## Inventory in a cloud environment

"No physical servers" does not make hardware inventory N/A. List the virtual components that
provide equivalent functions, noting that they are cloud-managed and shared:

- compute (container tasks, VMs);
- load balancers;
- network security controls (WAF, managed firewall, security groups);
- routing/egress (NAT, gateways, peering);
- networks (virtual networks, subnets);
- data stores (managed databases, with engine and version);
- secrets management;
- identity and access (roles, policies);
- security tooling (threat detection, audit logging, vulnerability scanning, EDR);
- endpoints (managed laptops).

For cloud infrastructure-as-a-service, the provider's own PCI responsibilities are evidenced
by their Attestation of Compliance (see `docs/service-providers.md`), and you document your
side in the inventory and the shared-responsibility matrix.

## Keeping it audit-ready

- Generate the software inventory from the source of truth (package manifests, IaC), not by
  hand.
- Keep the network diagram in version control next to the IaC so it cannot silently drift.
- At scope review, walk the data-flow diagram first; the inventory and diagrams should fall
  out of it consistently. Inconsistencies between the three are the most common finding.

See `templates/scope-inventory.template.md` for a starting structure.
