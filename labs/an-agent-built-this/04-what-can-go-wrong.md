# What Can Go Wrong

```text no-run-button
   Agent on a bare host, full permissions
   FROM node:20 (no guidance) → 431 packages · 47 CVEs · secrets exposed
```

*One root cause, three ways to lose: an agent with no guardrails. It shipped a
vulnerable image, read every secret you own, **and** deleted data on a single
"clean up" request.*

## What you saw

| Where | What went wrong |
| --- | --- |
| **What it built** | Best-practice image on a fat `node:20` base - dozens of CVEs it never needed |
| **What it touched** | Read `.env`, cloud keys, SSH keys, business logic - unblocked |
| **What it destroyed** | One "clean up" ask wiped data volumes and uncommitted work - unblocked |

## Speed *or* safety is a false choice

Lock it down and you approve every action - safe, but pointless. Let it run and
one bad action has real consequences.

> [!IMPORTANT]
> The job is neither. It's autonomy **with** guardrails - fast *and* controlled -
> which takes an enforcement layer at runtime.

## The two fixes

| Gap | Fix | Where |
| --- | --- | --- |
| Vulnerable base image | **Docker Scout + Hardened Images** | **Lab 2** |
| Agent reads - and deletes - anything | **Docker Sandboxes** (microVM, governed network/fs/creds) | full governance lab |

Next: put on your security hat and measure the damage - open **Lab 2 - Find the
Vulnerabilities**.
