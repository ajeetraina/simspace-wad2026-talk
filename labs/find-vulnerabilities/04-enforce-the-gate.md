# Enforce It: The Policy Gate

```text no-run-button
   docker scout policy
   FROM node:20  → no-critical-cves FAILED  ✗ blocked
   FROM dhi …    → no-critical-cves PASSED  ✓ promoted
```

*Hardening once is good. Making the vulnerable version **impossible to ship** is
governance. A Scout policy gate turns "use a hardened base" into a check that
fails closed.*

## Step 1 - Evaluate the gate

```bash
docker scout policy product-catalog:latest
```

Hardened → all three checks **PASS** (no critical/high CVEs, SBOM, provenance)
and the image may be promoted. Still on `node:20` → they **FAIL**, and CI refuses
to push it.

## Step 2 - Why this is the point

- A developer **or an agent** can't merge a vulnerable image past the gate.
- Same Scout data you checked by hand - no gap between local and CI.
- "Use a hardened base" becomes a **check on every build**, not a wiki page.

## What you accomplished

| Step | Result |
| --- | --- |
| Scanned the agent's image | 47 CVEs, 2C 12H - most from the base |
| Swapped to DHI | One line, distroless + signed base |
| Re-scanned | 2 CVEs (0C 0H), 335 fewer packages |
| Added a policy gate | Vulnerable images fail closed |

> [!IMPORTANT]
> Two layers, one story: harden the supply chain with **Scout + DHI**, contain
> the runtime with **Docker Sandboxes + AI Governance**. Together, agents run fast
> *and* safely.

For the full runtime story - microVM sandbox, network/filesystem/credential
isolation, MCP governance, audit - open the **Securing the Agentic Stack** lab.
