# You Can't Govern What You Can't See

```text no-run-button
   "clean build" ≠ "clean image"
   multi-stage · non-root  ✓   but  FROM node:20 → 431 packages, ??? CVEs
```

*In Lab 1 an agent containerised the app well. But a good Dockerfile on a fat
base still ships whatever CVEs live in that base. The image looks clean - only a
scanner can say if it **is**.*

## Where we left off

```bash
cat product-catalog-demo-showcase/Dockerfile
```

Line 1: `FROM node:20` - a full Debian userland with shells and libraries the app
never calls. The agent picked it with no guidance; a human would ship the same.

## Your job

| Step | Do |
| --- | --- |
| **Scan** | Docker Scout - real CVE count and where it comes from |
| **Harden** | Swap to a Docker Hardened Image (DHI) |
| **Re-scan** | Watch the CVEs collapse |
| **Enforce** | A policy gate that fails closed |

> [!NOTE]
> This is a **supply-chain** problem, not a Gordon/Claude problem. The fix isn't
> "write a better Dockerfile" - it's "start from a better base," proven by a scan
> and required by a gate.

Next: **Scan It with Docker Scout**.
