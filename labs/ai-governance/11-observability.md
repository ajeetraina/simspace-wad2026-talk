# Observability & Audit

```text no-run-button
   Product Catalog agent  ──▶  sbx daemon (every decision)  ──▶  audit log (JSONL)
     network · mounts · tools       allow / deny + why           user · org · session
                                                                  └─▶ Splunk / Datadog / SIEM
```

*Throughout this lab you governed one agent: the one building and running the **Product
Catalog**. This last section shows the trail it left - every request its sandbox allowed
or blocked, and who was behind it. Enforcement you can't see isn't something a security
team will trust.*

## See what the agent did

`sbx policy ls` shows the *rules*. `sbx policy log` shows what they actually *did* - the
real requests the Product Catalog agent made from inside its sandbox:

```bash
sbx policy log
```

There's the Product Catalog agent's whole session, one line per destination and all under
the `catalog` sandbox: `api.anthropic.com` - the agent reasoning about the catalog's
Dockerfile - was **allowed** (allow AI services); a prompt-injected attempt to POST the
catalog's **Stripe key** to `paste.ee` was **blocked** (deny exfiltration); and a reach
for an unlisted host was **blocked** by default-deny. Same agent, every decision
attributed to the rule behind it. For scripting, emit JSON:

```bash
sbx policy log --json
```

## Who did it - the audit trail

The traffic log tells you *what* and *why*, but not *who*. Docker AI Governance also writes
a sealed audit event per decision - **with the signed-in user, org, and session on every
record**. Read the Product Catalog agent's audit trail:

```bash
sbx audit log
```

Its blocked attempt to leak the catalog's **Stripe key** to `paste.ee` shows up as one
sealed record, stamped with the user, org, and session:

```json no-run-button
{
  "timestamp": "2026-05-28T19:15:00Z",
  "decision": "AUDIT_DECISION_DENY",
  "username": "jordandoe",
  "org_name": "$$org$$",
  "sandbox": "product-catalog",
  "resource_id": "paste.ee:443",
  "deny_reason": ["deny exfiltration"],
  "action_type": "network_egress"
}
```

Point Splunk, Datadog, or Sentinel at these `*.jsonl` files and you have a per-developer,
per-decision trail of exactly what every agent tried.

> [!IMPORTANT]
> Audit logging is a **paid** part of Docker AI Governance and only activates when
> `$$org$$` enforces a centralized policy. No governance → no audit records.

## Governance-as-code

Everything you did in the Admin Console has an equivalent on the
[**AI Governance API**](https://docs.docker.com/reference/api/ai-governance/) - ideal for
version control and CI. The `setup-policies.sh` helper wraps those API calls so you can
provision the whole org policy in one shot:

```bash
bash setup-policies.sh network
```

Console and API write to the **same** source of truth for `$$org$$` - pick whichever fits
your workflow.

## What you built

You took the blast radius an ungoverned agent has over the Product Catalog - reaching the
network, reading secrets, holding a live key, calling any tool - and closed **every
boundary**: network, filesystem, credential, and MCP, with one policy engine that fails
closed and leaves an audit trail.

- **Define once** - Admin Console or Governance API
- **Enforce everywhere** - synced to every developer, un-overridable locally
- **See everything** - the traffic log plus a SIEM-ready audit trail with user attribution

That's the defensible, end-to-end enforcement story for the Product Catalog agent - one you
can now walk a security team through. 🎉

Learn more at [docker.com/products/ai-governance](https://www.docker.com/products/ai-governance/).
