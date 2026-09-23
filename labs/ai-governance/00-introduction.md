# Why Secure the Agentic Stack

```text no-run-button
   An agent builds it        You harden what it SHIPS        You contain what it TOUCHES
   ┌────────────────┐        ┌──────────────────────┐        ┌──────────────────────────┐
   │ Gordon          │       │ Docker Scout           │       │ Sandbox policies          │
   │ (docker ai)     │  ──▶  │ Docker Hardened Images │  ──▶  │ network · filesystem ·    │
   │ containerises   │       │ signed, gated CI       │       │ credentials · MCP tools   │
   └────────────────┘        └──────────────────────┘        └──────────────────────────┘
        the work                  supply chain                     blast radius
```

*An AI agent now makes decisions that used to pass through human review - which base
image to build on, which dependencies to pull, which tools to call, what to read on
your disk. This lab secures **both** ends of that: the artifact it **ships** and the
blast radius it **touches**.*

AI agents - Gordon, Claude, Copilot, Cursor, custom MCP servers - run with the
**same blast radius as the developer running them**: your filesystem, your secrets,
your network. And whatever they build inherits whatever base image they happened to
pick. That's fine when the agent does what you expect. It's a disaster when:

- The agent ships an image on a fat base full of CVEs, straight to production
- A prompt-injected agent uploads SSH keys to `paste.ee`
- A coding agent reads your `.env` and posts your API key to a model endpoint
- A misconfigured MCP server exfiltrates source code to an unknown destination

"Don't run autonomous agents" doesn't scale - developers want them, and they
deliver. The answer is **governance**: *harden what the agent produces*, and
*contain what the agent can reach*. That's what you'll build here, end to end.

> [!NOTE]
> Everything in this lab is **simulated** - no real Docker, `sbx` daemon, or
> network. Every learner sees the exact same allow/deny decisions, with nothing
> to install. The commands, outputs, and policy behaviour mirror real
> [Docker Hardened Images](https://www.docker.com/products/hardened-images/),
> [Docker Scout](https://www.docker.com/products/docker-scout/), and
> [Docker AI Governance](https://www.docker.com/products/ai-governance/).

## Set your organization

Most commands and links below substitute `$$org$$` for your Docker Hub org. Set it
once here:

:variableDefinition[org]{prompt="Which Docker Hub organization will you use?"}

## The two halves of this lab

| Half | Question it answers | Sections |
| --- | --- | --- |
| **Harden what it ships** | Is the artifact safe to run? | Gordon → Docker Scout → Hardened Images → Sign & Gate (CI) |
| **Contain what it touches** | Can the agent exceed its scope? | Policy Model → Network → Filesystem → Credential → MCP |

## What this lab covers

| Section | What you'll do |
| --- | --- |
| **Gordon Containerises It** | **Start here** - Gordon (`docker ai`) containerises a real app with best practices |
| Docker Scout | Scan the image; the fat base drags in dozens of CVEs |
| Hardened Images (DHI) | Swap the base, re-scan - the CVEs collapse to near-zero |
| Sign It, Then Gate It (CI) | A pipeline that signs the image and fails closed on policy |
| The Policy Model | How org policies flow to every developer's sandbox |
| Network Enforcement | Contain egress - three `curl`s, three outcomes |
| Filesystem Isolation | The agent can't read or mount your secrets (403 at creation) |
| Credential Isolation | The real API key never enters the sandbox |
| MCP Governance | Register servers behind one governed gateway; gate tools with Cedar |
| Putting It All Together | **The capstone** - one rogue agent, four attacks, one policy engine |
| Observability & Audit | The visibility half, plus governance-as-code (the API) |

By the end you'll have a defensible, end-to-end story you can walk a security team
through. Head to **Gordon Containerises the Product Catalog** - where an agent does
something real, and shows you exactly what needs securing.
