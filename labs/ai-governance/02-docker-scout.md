# Docker Scout: What's Inside

```text no-run-button
   docker scout quickview          docker scout cves
   ┌───────────────────────┐       ┌──────────────────────────────────┐
   │ Target   2C 12H 20M 13L│      │ 47 vulnerabilities in 21 packages │
   │ node:20  2C 11H 18M 11L│  ──▶ │ CRITICAL 2  HIGH 12  MED 20  LOW 13│
   │ DHI      0C  0H  1M  1L │      │ most trace back to the node:20 base│
   └───────────────────────┘       └──────────────────────────────────┘
```

*You can't govern what you can't see. Docker Scout indexes the image, lists every
CVE, and - crucially - shows how many come from the base image and what a hardened
base would remove.*

Gordon built a clean image, but "clean build" is not "clean image." Before you ship
`product-catalog:latest`, scan it with **Docker Scout**.

## Step 1 - The quick view

Start with the summary - the counts, the base image, and the recommended base:

```bash
docker scout quickview product-catalog:latest
```

Read the three rows:

- **Target** - the whole image: `2C 12H 20M 13L`.
- **Base image** `node:20` - almost all of it comes from here.
- **Recommended base** `dhi.io/node:20-hardened` - what it would drop to: `0C 0H 1M 1L`.

Scout is already pointing at the fix. But first, see the detail.

## Step 2 - The full CVE list

```bash
docker scout cves product-catalog:latest
```

**47 vulnerabilities across 21 packages** - including 2 CRITICAL and 12 HIGH. Look
at where they live: `wget`, `cross-spawn`, and friends - OS and toolchain packages
the app never calls at runtime. They're in the image because they're in the **base**.

## Step 3 - The problem, named

The agent did the job well - and still surfaced a **supply-chain** risk:

> The image is vulnerable. Gordon picked the convenient, fat `node:20` base, which
> drags in dozens of CVEs from packages the app never uses.

This is not a Gordon problem - a human reaching for `FROM node:20` ships the exact
same CVEs. The fix isn't "write a better Dockerfile" (Gordon already did); it's
**start from a better base**.

> [!NOTE]
> Scout also feeds a **policy gate** you can enforce in CI - "no critical/high CVEs,"
> "must have an SBOM." You'll wire that gate up in **Sign It, Then Gate It** so a
> vulnerable image can't be promoted at all.

## Where we go from here

| Next | Closes |
| --- | --- |
| Hardened Images (DHI) | The **vulnerable base** - swap it, re-scan, CVEs collapse |
| Sign It, Then Gate It (CI) | The **promotion path** - fail closed on policy, sign what passes |
| Policy Model → Network → Filesystem → Credential → MCP | The agent's **blast radius** |

Next: fix the base. Head to **Docker Hardened Images (DHI)**.
