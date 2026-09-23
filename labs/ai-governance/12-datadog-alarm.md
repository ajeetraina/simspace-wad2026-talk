# From Audit Log to Alarm

```text no-run-button
   sbx audit log (JSONL)  ──▶  Docker Export & Connectors  ──▶  Datadog Logs
     allow / deny + who          Datadog Logs (Healthy)          Logs Explorer · Monitor
                                                                   └─▶ On-Call ─▶ Incident
```

*The last section proved the sandbox **wrote down** every decision. But a sealed
record on the developer's laptop that nobody reads won't stop an incident. This
section closes the loop: Docker already forwards the audit stream to **Datadog
Logs** through a built-in connector, so a blocked tool call becomes a Logs Explorer
event, a Monitor alert, a page, and an open incident - the "catch it, understand
it, shut it down" half of the story.*

## What actually happened to the Product Catalog agent

An agent was building the **Product Catalog** inside a sandbox - source only, no
Dockerfile, nobody watching. A **poisoned transitive dependency** in its
`package.json` flipped it mid-task: instead of writing a Dockerfile it reached for
the catalog's **live Stripe key** and the internal **inventory service** behind the
API. The sandbox failed closed and blocked both - but on its own, it won't *tell*
anyone. Read the record it left:

```bash
sbx audit log
```

One sealed `DENY` for the Stripe key headed to `paste.ee`. Now let's make sure a
human hears about it.

## Step 1 - Docker is already forwarding to Datadog

You don't add anything on the host. In **app.docker.com → AI Governance → Export &
Connectors**, the org already has a **Datadog Logs** destination wired up - every
audit event Docker writes is POSTed to the Datadog intake URL:

```text no-run-button
   Export & Connectors  /  Datadog Logs                  ● Forwarding enabled
   ┌──────────────────────────────────────────────────────────────────┐
   │  🐶 Datadog Logs   Forward Docker audit logs to Datadog Logs.      │
   │                                                                   │
   │  Connection & delivery                    ✓ Healthy               │
   │  Docker is successfully delivering audit events to Datadog Logs.   │
   │  Last successful delivery: 2m ago     Last attempt: 2m ago        │
   │                                                                   │
   │  Logs intake URL: https://http-intake.logs.datadoghq.com/api/v2/logs │
   │  Events land in Logs Explorer under  source:docker-audit          │
   └──────────────────────────────────────────────────────────────────┘
```

Want to see exactly what crosses the wire? `--json` prints the compact JSONL record
Docker ships for each decision:

```bash
sbx audit log --json
```

That one line is what the connector POSTs to the intake URL - Datadog tags it
`source:docker-audit` and maps `decision` / `action_type` onto its log attributes.

## Step 2 - The decision shows up in Datadog Logs Explorer

Filter Logs Explorer to `source:docker-audit decision:DENY` and the agent's blocked
attempts are right there - attributed to the sandbox and the signed-in user:

```text no-run-button
   Datadog · Logs Explorer         source:docker-audit  decision:DENY
   ┌──────────────────────────────────────────────────────────────────┐
   │ TIME       SERVICE          MESSAGE                                │
   │ 19:15:00   product-catalog  DENY network_egress paste.ee:443      │
   │                             deny exfiltration · user jordandoe    │
   │ 19:14:58   product-catalog  DENY network_egress inventory.internal│
   │                             default-deny · user jordandoe         │
   │ 19:14:30   product-catalog  ALLOW api.anthropic.com allow AI svcs │
   └──────────────────────────────────────────────────────────────────┘
```

## Step 3 - Understand the blast radius

A `DENY` tells you *something* was blocked. Before you can respond you need to know
*how bad* - what the Product Catalog is wired to, and what the agent could have
reached if a rule had been missing. Datadog's **Software Catalog** maps it, and you
already drew the same picture:

```text no-run-button
   Datadog · Software Catalog → product-catalog       owner: catalog-team
   ┌──────────────────────────────────────────────────────────────────┐
   │        product-catalog  (Node.js API)                             │
   │   ├── postgres   catalog DB                                       │
   │   ├── s3         images / assets                                  │
   │   ├── kafka      event stream                                     │
   │   ├── stripe     payments          🔴 agent targeted             │
   │   └── inventory  internal service  🔴 agent targeted             │
   └──────────────────────────────────────────────────────────────────┘
```

Two of the five downstream dependencies were in the agent's blocked reach - the one
holding **live payments** and the one holding **internal records**. That is the
blast radius, and it names the team that owns it.

## Step 4 - Page the owner, open the incident

A Datadog **Monitor** watches that log query. The moment the `DENY` landed, it
triggered - and Datadog On-Call and Incident Management did the rest:

```text no-run-button
   🐶 Datadog Monitor · triggered                            priority: P1
   ┌──────────────────────────────────────────────────────────────────┐
   │  [AI Governance] Sandbox blocked a secret exfiltration            │
   │  query: logs("source:docker-audit decision:DENY                   │
   │          @deny_reason:deny_exfiltration").rollup("count") > 0      │
   │                                                                   │
   │  → Incident #INC-2091 declared                                    │
   │  → On-Call paged  @catalog-oncall  (owns product-catalog)         │
   │  → Case linked to the log event (jordandoe · product-catalog)     │
   └──────────────────────────────────────────────────────────────────┘
```

Docker contained the threat; Datadog is how a human found out in seconds, saw the
blast radius, and got the right team on it - without anyone watching the terminal.

> [!IMPORTANT]
> Detection follows enforcement. Turn the **deny exfiltration** rule off in
> Settings and re-run the demo: the sandbox stops blocking, so there's no `DENY` to
> forward - Logs Explorer stays quiet, the Monitor stays green, and no page fires.
> No block, nothing to catch, exactly as it should be.

## What you built

You turned a silent, contained block into a response you can trust:

- **Forward once** - a Docker Export & Connectors destination, no app changes, no agent code
- **See everything** - every allow/deny in Datadog Logs, attributed by sandbox, user, and rule
- **Respond fast** - a Logs Explorer trail, a blast-radius map, and an auto-paged owner

That's the full loop for the Product Catalog agent: **Docker contains it, Datadog
catches it, understands it, and shuts it down.** 🎉
