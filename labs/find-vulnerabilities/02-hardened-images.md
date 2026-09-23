# Docker Hardened Images (DHI)

```text no-run-button
   FROM node:20                →   FROM dhi.io/node:20-hardened
   47 CVEs (2C 12H 20M 13L)         2 CVEs (0C 0H 1M 1L)
   fat base, 431 packages           minimal · signed · SBOM + SLSA provenance
```

*The Dockerfile is fine; the **base** is the liability. A Docker Hardened Image
is a minimal, continuously-patched, signed base with an SBOM and SLSA provenance
built in.*

## Step 1 - Switch the base

Ask the agent to swap it and rebuild - `-dev` variant to build, distroless
`-hardened` at runtime:

```bash
docker ai "Switch the Dockerfile to a Docker Hardened Image base and rebuild"
```

```bash
cat product-catalog-demo-showcase/Dockerfile
```

> [!NOTE]
> By hand: edit the two `FROM` lines to `dhi.io/node:20-dev` and
> `dhi.io/node:20-hardened`, then `docker build -t product-catalog:latest .`

## Why DHI is a governance story

- **Minimal** - no shells or package managers for an agent (or attacker) to abuse.
- **Signed + attested** - signature, SBOM, and SLSA provenance you can verify.
- **Continuously patched** - fixed at source and rebuilt, so you inherit the fix.

That makes "use a hardened base" something you can **require**, not just
recommend - if it's *enforced* (last section).

Next: prove the swap worked - **Re-scan: The Collapse**.
