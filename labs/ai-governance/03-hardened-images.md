# Docker Hardened Images (DHI)

```text no-run-button
   product-catalog:latest              product-catalog:latest (rebuilt)
   FROM node:20                        FROM dhi.io/node:20-hardened
     47 CVEs  (2C 12H 20M 13L)   ──▶     2 CVEs  (0C 0H 1M 1L)
     fat base, unused packages          minimal · signed · SBOM + SLSA provenance
```

*Scout showed almost every CVE traces to the fat base. Swap in a **Docker Hardened
Image**, rebuild, and re-scan: the vulnerabilities collapse. Same app, a fraction of
the attack surface - and it arrives signed and attested.*

The Dockerfile is already best-practice; the base is the liability. The fix is a
**Docker Hardened Image (DHI)** - a minimal, continuously-patched, signed base with
an SBOM and SLSA provenance built in.

## Step 1 - Recall the damage

```bash
docker scout cves product-catalog:latest
```

47 findings, most from the base. The app code is fine; the **base** is the problem.

## Step 2 - Have Gordon switch to a hardened base

Ask Gordon to swap the base and rebuild - it keeps the multi-stage layout, using the
DHI `-dev` variant to build and the distroless `-hardened` variant at runtime:

```bash
docker ai "Switch the Dockerfile to a Docker Hardened Image base and rebuild"
```

See the change - note the two DHI stages:

```bash
cat product-catalog-demo-showcase/Dockerfile
```

## Step 3 - Re-scan

```bash
docker scout cves product-catalog:latest
```

The CVE count collapses to **2 (0C 0H 1M 1L)** - the critical and high findings are
gone, because the hardened base ships only what the app needs and is patched
upstream. Same application, a fraction of the attack surface.

## Why DHI is a governance story, not just a smaller image

A Docker Hardened Image isn't only "fewer CVEs." Each one is:

- **Minimal** - no shells, package managers, or unused libraries for an agent (or
  an attacker) to abuse.
- **Signed + attested** - a cryptographic signature, an SBOM, and SLSA provenance
  you can *verify* before it runs.
- **Continuously patched** - CVEs are remediated at the source and the image is
  rebuilt, so you inherit the fix.

That makes "use a hardened base" something an org can **require**, not just
recommend. But a recommendation only helps if it's *enforced*.

> [!NOTE]
> Docker also ships a **DHI MCP server** that exposes hardened-image metadata -
> CVEs, SBOMs, attestations, mirrors - as governed tools. You'll register it and
> apply a read-only-vs-mutating policy to it in **MCP Governance**.

## What you fixed - and what's next

The image is now hardened, but nothing yet **stops** a vulnerable image from being
pushed. That's the job of a pipeline gate:

| Next | What it does |
| --- | --- |
| **Sign It, Then Gate It (CI)** | A pipeline that fails closed on policy and signs what passes |

After that, we switch halves - from *what the agent builds* to *what the agent can
touch*: **The Policy Model** and the sandbox. Head to **Sign It, Then Gate It**.
