# Scan It with Docker Scout

```text no-run-button
   quickview          cves
   Target  2C 12H 20M 13L   →   47 vulnerabilities in 21 packages
   node:20 2C 11H 18M 11L        most trace back to the node:20 base
   DHI     0C  0H  1M  1L
```

*Scout indexes the image, lists every CVE, and shows how many come from the base
and what a hardened base would remove.*

## Step 1 - Quick view

```bash
docker scout quickview product-catalog:latest
```

Three rows: **Target** `2C 12H 20M 13L`, **base** `node:20` (where almost all of
it comes from), **recommended base** `dhi.io/node:20-hardened` → `0C 0H 1M 1L`.

## Step 2 - Full CVE list

```bash
docker scout cves product-catalog:latest
```

**47 vulnerabilities, 2 CRITICAL and 12 HIGH.** They live in `wget`,
`cross-spawn`, and friends - packages the app never calls. They're here because
they're in the **base**.

## Step 3 - Ask Scout what to do

```bash
docker scout recommendations product-catalog:latest
```

It names the one high-leverage change: swap `node:20` for
`dhi.io/node:20-hardened`.

> [!NOTE]
> Scout also feeds a **CI policy gate** - "no critical/high CVEs," "must have an
> SBOM." You'll wire that up in the last section.

Next: fix the base - **Docker Hardened Images (DHI)**.
