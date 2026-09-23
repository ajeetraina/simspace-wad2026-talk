# Re-scan: The Collapse

```text no-run-button
   BEFORE node:20   47  (2C 12H 20M 13L) · 431 pkgs
   AFTER  DHI        2  (0C  0H  1M  1L) ·  96 pkgs
```

*Same app, one line changed - and the vulnerabilities collapse.*

## Step 1 - Re-scan

```bash
docker scout cves product-catalog:latest
```

Down to **2 (0C 0H 1M 1L)**. The critical and high findings are **gone** - the
hardened base ships only what the app needs, patched upstream.

## Step 2 - Confirm

```bash
docker scout quickview product-catalog:latest
```

No recommended-base row: you're already on it.

## Step 3 - Before/after

```bash
docker scout compare product-catalog:latest --to product-catalog:node20
```

| Category | node:20 | DHI | Change |
| --- | --- | --- | --- |
| Critical | 2 | 0 | **-2** |
| High | 12 | 0 | **-12** |
| Packages | 431 | 96 | **-335** |

45 fewer vulnerabilities, 335 fewer packages - **no application code changed**.

> [!IMPORTANT]
> The base image is a **supply-chain decision**. An agent (or human) grabbing the
> convenient base makes it for you. Scout + a hardened base put it back in your
> hands.

Next: make the vulnerable version impossible to ship - **Enforce It: The Policy
Gate**.
