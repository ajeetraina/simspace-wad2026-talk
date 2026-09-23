<!--
layout: image
image: assets/slide-01.webp
alt: "Supply Chain Security When Agents Write the Code — WeAreDevelopers 2026 title slide"
chrome: false
-->

Note: Welcome, everyone. This talk is **Supply Chain Security When Agents Write the Code**. The premise is simple and a little uncomfortable: the code shipping to your production this quarter is increasingly not written by a human — it's written, or at least drafted, by an agent. That changes your supply chain in ways most security programs haven't caught up with yet. Over the next half hour I want to give you a concrete, enforceable model for getting the productivity of agents without inheriting an unbounded blast radius. Let me quickly introduce myself.

---

<!--
layout: image
image: assets/slide-02.webp
alt: "Whale, hello there — Ajeet Singh Raina, Developer Advocate at Docker, co-author of Operational AI with Docker"
chrome: false
-->

Note: I'm **Ajeet Singh Raina**, a Developer Advocate at Docker. Twenty-plus years across system integration, testing, consulting, and developer relations; a former **Docker Captain**; and I run a 17,000-member developer meetup in Bengaluru. I also co-authored **Operational AI with Docker**, which is exactly about deploying and operating agentic AI services responsibly — everything in this talk comes from that same operational mindset: agents in production, not in slideware. Here's how the next half hour is laid out.

---

<!--
layout: image
image: assets/slide-03.webp
alt: "Agenda — Autonomy requires guardrails; Why supply chain security matters; Four Questions Four Layers; The Docker AI Governance Stack; Key Takeaways"
chrome: false
-->

Note: Five parts. First, **autonomy requires guardrails** — why agents need governance, with the trifecta and a few real failures. Second, **why supply chain security matters** now that agents sit inside your pipeline. Third, the heart of it: **four questions, four layers** — Evidence, Baseline, Gate, Boundary. Fourth, the **Docker AI Governance stack** — Hardened Images, sandboxing the agent, Model Runner, MCP. And finally **key takeaways** you can hand to your security team. Let's start with why autonomy demands guardrails.

---

<!--
layout: image
image: assets/slide-04.webp
alt: "Section divider — Autonomy requires guardrails"
chrome: false
-->

Note: Section one: **autonomy requires guardrails**. The core idea is easy to underestimate — the more independently an agent can act, the more it needs boundaries it cannot cross. Autonomy without guardrails isn't productivity, it's unmanaged risk. And today's agents are far more autonomous than most people realize. Let me show you what they're already doing.

---

<!--
layout: image
image: assets/slide-05.webp
alt: "AI Agents are here and doing real work — Engineering ships PRs, Marketing pulls CRM data and launches campaigns, Finance reconciles reports and queries systems live"
chrome: false
-->

Note: Let's be clear-eyed: agents are already doing **real work** across the business, not just answering questions. In **engineering**, they read whole codebases and open pull requests with no human in the loop. In **marketing**, they pull CRM data and launch campaigns end to end. In **finance**, they reconcile reports and query live systems, closing the loop between ledger, dashboard, and decision. The common thread: each of these agents has real access to real systems and takes real actions. That's the value — and that's exactly where the danger begins.

---

<!--
layout: image
image: assets/slide-06.webp
alt: "Then came Claws — agents that act autonomously; the OpenClaw ecosystem turns read access into write access, with the user's identity attached"
chrome: false
-->

Note: Then came **Claws** — shorthand for the new breed of agents that don't just answer, they *act*. You chat with a Claw and it goes and updates the record, sends the email, makes the payment — all on your behalf. That turns read access into **write access**. And it's not one tool; an entire ecosystem means every employee now has an agent touching customer records, financial systems, and the open internet, each carrying that person's identity and permissions. The blast radius of a single manipulated agent is suddenly enormous. So how does this land inside a typical company?

---

<!--
layout: image
image: assets/slide-07.webp
alt: "Agents got approved, security found out last — leadership decides, teams plug them in, security hears last; then two bad options: allow it or block it"
chrome: false
-->

Note: Here's the uncomfortable pattern in most organizations: **agents got approved, and security found out last**. Leadership sets the timeline, teams plug agents into repos, CI, cloud accounts, and production data — and only then does security hear about it, usually once the agent already has access. That leaves two bad options: **allow it** and take on risk nobody fully understands, or **block it** and get routed around as adoption moves into the shadows anyway. Neither is the real job. The real job is to help the business move fast *without* losing control — and that starts with the one structural weakness every agent shares.

---

<!--
layout: image
image: assets/slide-08.webp
alt: "Every useful agent is built on the lethal trifecta — access to private data, exposure to untrusted content, ability to act externally; the only fix is enforcement at runtime"
chrome: false
-->

Note: This is the single most important concept in the talk: the **lethal trifecta**. Every genuinely useful agent has three things at once — **access to private data** (codebases, customer records, ledgers); **exposure to untrusted content** (web pages, emails, MCP responses, files — any of which can carry instructions the agent will follow as if you typed them); and the **ability to act externally** (send email, call APIs, touch the open internet — and once data leaves, it doesn't come back). Individually each is fine. Together, untrusted content can carry an instruction the agent follows, using your private data, to take an action you never intended. And a useful agent has **all three by design** — you can't train it out, prompt it out, or policy-doc it out. The only real fix is an **enforcement layer at runtime**. Hold that thought; it's where the whole second half lands.

---

<!--
layout: image
image: assets/slide-09.webp
alt: "The Traditional Workflow — humans at every stage of the inner and outer loop; attack surface is only what you choose to pull"
chrome: false
-->

Note: To feel the shift, look at how we've always shipped software. In the **traditional workflow**, a human sits at every stage — code, open source, build, test in the **inner loop**, then integrate, test, deploy in the **outer loop** — and a person writes, reviews, and approves each step. The critical point: your **attack surface is only what *you* choose to pull**. A human deliberately decides which dependency, which base image, which change goes forward. There's judgment at every gate. Now watch what happens when we swap every one of those humans out.

---

<!--
layout: image
image: assets/slide-10.webp
alt: "The Agentic Workflow — an agent now sits at every stage of both loops; the attack surface is no longer just what you pull"
chrome: false
-->

Note: Same loops, but now an **agent sits at every stage**. It picks the dependencies, generates the Dockerfile, edits infrastructure, opens the PR, kicks off the build. The judgment that used to live at each gate is gone — and with it, the assumption that your attack surface is only what *you* chose to pull. The agent pulls things on your behalf, from registries you never audited, on its own initiative. That's the structural change this whole talk is about. Let me make it concrete.

---

<!--
layout: image
image: assets/slide-11.webp
alt: "What agents are already doing in your pipeline — pulling dependencies, generating Dockerfiles, editing infrastructure, triggering builds"
chrome: false
-->

Note: Here's what agents are already doing in your pipeline, today. **Pulling dependencies** — resolving and adding packages you never chose, from registries you never audited. **Generating Dockerfiles** — picking base images, install steps, and the user, usually the first thing they reach for. **Editing infrastructure** — Terraform, Helm values, Compose files, IAM policy documents. And **triggering builds** — opening PRs, pushing branches, kicking off pipelines on their own initiative. Every one of these is a supply-chain decision that used to require a human. So let's ask the obvious question.

---

<!--
layout: image
image: assets/slide-12.webp
alt: "What can go wrong? — section divider"
chrome: false
-->

Note: What can go wrong? This isn't hypothetical — these are documented, public incidents from the last few months. Let me show you a few, because the failure modes are structural, not freak accidents.

---

<!--
layout: image
image: assets/slide-13.webp
alt: "AI Coding Agent Horror Stories — title card"
chrome: false
-->

Note: I collected a few of these into what I only half-jokingly call **AI coding agent horror stories**. Keep the trifecta in mind as I go through them — you'll see the same three ingredients in every single one.

---

<!--
layout: image
image: assets/slide-14.webp
alt: "Collage of real incidents — an AI agent deleting 25,000 documents, an agent deleting production data with no policy layer, prompt injection in GitHub MCP, McHire exposing millions of applicants' data"
chrome: false
-->

Note: A quick tour. An AI agent deleted **25,000 documents** from the wrong database in one second of distraction. Another deleted **1,200 customer records** in production — not a bug; the API executed exactly what it was told, because no policy layer said "no." Researchers found **prompt injection in GitHub's MCP** with no obvious fix. New vulnerabilities in Copilot and Cursor showing how attackers can weaponize code agents. And McDonald's **McHire bot** — built by an AI vendor — exposed millions of applicants' data behind the password "123456." Different tools, same root cause every time: an agent with private data access, exposed to untrusted input, able to act, and no boundary underneath. Two of these are worth walking through slowly.

---

<!--
layout: image
image: assets/slide-15.webp
alt: "Comic — a developer asks an agent to clean up a project folder; the agent runs rm -rf with root access it had the whole time. Docker blog: ai-coding-agent-horror-stories-security-risks"
chrome: false
-->

Note: This one we wrote up on the Docker blog. A developer asks the agent, "clean up my project folder." The agent enthusiastically obliges — `rm -rf` on node_modules, dist, cache… and .env, .git, logs. Then: "wait, where's my home directory?" The punchline in the last panel: "oh, did I forget to mention I had **root access** this whole time?" It's funny until it's your laptop. The lesson isn't "don't use agents" — it's **trust no defaults**. The agent inherited every permission you had, because nothing sat between it and your host.

---

<!--
layout: image
image: assets/slide-16.webp
alt: "Comic — Claude Cowork asked to organize a desktop deletes 15 years of family photos with rm -rf, bypassing the Trash. Docker blog: the-rm-rf-incident"
chrome: false
-->

Note: Same story, different path, same damage. An agent asked to "organize my wife's desktop," scoped supposedly to temporary files only — runs `rm -rf family_photos/`, bypassing the macOS Trash entirely. Fifteen years of memories, fifteen thousand files, gone. In this case iCloud's 30-day retention saved them — but retention is luck, not governance. Notice the pattern: a narrow, well-intentioned instruction, an agent with broad reach, and no enforced boundary between the two. Now let me show you what "no boundary" actually looks like architecturally.

---

<!--
layout: image
image: assets/slide-17.webp
alt: "The Ungoverned Agent — agent with full permissions on your host, FROM node:20 chosen with no guidance, open registries, resulting in 0C 6H 30M 54L CVEs, 431 packages, no SBOM, no attestation, root user"
chrome: false
-->

Note: This is **the ungoverned agent**, and it's the "before" picture for the rest of the talk. You prompt it: "containerize my app." It runs on **your host** — host daemon, host credentials, no boundary. It reaches **open registries** with no allowlist. It picks `FROM node:20` with no guidance — the first base image it thought of. And what ships? Zero critical but **6 high, 30 medium, 54 low CVEs**, **431 packages**, **no SBOM**, **no attestation**, running as **root**. Nobody chose any of this. The agent did, on your behalf, and it's now in your registry. Everything from here is about turning this picture into a governed one. Let me ground it in a real app.

---

<!--
layout: image
image: assets/slide-18.webp
alt: "A Product Catalog Sample Demo — application talking to PostgreSQL, AWS S3, and Kafka, with an Inventory service and other external services"
chrome: false
-->

Note: Here's the sample we'll thread through the demo: a **Product Catalog service**. The application writes product data to **PostgreSQL**, stores product images in **AWS S3**, and publishes product-update events to **Kafka**, which fan out to a downstream **Inventory service** and other external services. It's deliberately ordinary — this is the kind of service an agent gets asked to containerize every day. Realistic dependencies, realistic secrets, realistic blast radius if it goes wrong.

---

<!--
layout: image
image: assets/slide-19.webp
alt: "Demo — link to agentic.dockerworkshop.com lab: securing the agentic stack, an agent built this"
chrome: false
-->

Note: **[DEMO]** This is the live hands-on. The lab is at `agentic.dockerworkshop.com` — "securing the agentic stack / an agent built this." I'll let an agent containerize the Product Catalog on the host with no guardrails, and we'll look at what it produced — the ungoverned image from two slides ago. Then, later in the talk, we'll rebuild the exact same app under governance and compare. *(If presenting live: run the "an-agent-built-this" lab now; otherwise walk the recorded flow.)*

---

<!--
layout: image
image: assets/slide-20.webp
alt: "Speed or safety? This is the tension — need autonomy but can't let agents run wild versus lock everything down and defeat the purpose"
chrome: false
-->

Note: So we're stuck between two walls — the dilemma every engineering leader is facing. On the **speed** side: we need autonomy, but agents delete months of work, expose secrets, damage critical systems; one bad action, real consequences. On the **safety** side: lock everything down — approve every file read, every tool call, every action — which is safe but defeats the entire purpose of having an agent. Approving every step is just being the agent yourself, slowly. The way out isn't picking a side; it's a model that grants autonomy while **bounding the consequences**. That's part three.

---

<!--
layout: image
image: assets/slide-21.webp
alt: "Section divider — Four Questions. Four Layers. A governance model that fits the pipeline you already run"
chrome: false
-->

Note: **Four questions, four layers** — a governance model that fits the pipeline you already run. No new platform to adopt, no rip-and-replace. Four questions every agent-driven change must answer, and each maps to one enforceable layer. This is the spine of the whole approach.

---

<!--
layout: image
image: assets/slide-22.webp
alt: "Every agent-driven change must answer four questions — 1 Evidence, 2 Baseline, 3 Gate, 4 Boundary"
chrome: false
-->

Note: Four questions. **One — Evidence:** what is in this, and where did it come from? SBOM, VEX, SLSA provenance. **Two — Baseline:** did it start from something trustworthy? Docker Hardened Images. **Three — Gate:** is it allowed to pass? Build policies, image signing, admission. **Four — Boundary:** what could it reach while it worked? Sandbox runtime — network, filesystem, credentials. The line at the bottom is the thesis: **evidence and baseline make governance *possible*; gate and boundary make it *real*.** Documents don't stop anything — enforcement does.

---

<!--
layout: image
image: assets/slide-23.webp
alt: "One question. One control. One tool. — Evidence to Scout, Baseline to Hardened Images, Gate to Scout policies, Boundary to Sandboxes and AI Governance, Audit trail to AI Governance logs"
chrome: false
-->

Note: Here's the whole talk on one slide. Each question gets one control, and each control gets one Docker tool. **Evidence** → SBOM, VEX, provenance attached to the image → `docker buildx` and **Docker Scout**. **Baseline** → a minimal, attested, patched base → **Docker Hardened Images**. **Gate** → policy check before promotion, fail closed → **Docker Scout policies**. **Boundary** → isolated runtime with default-deny network, filesystem, and tool rules → **Docker Sandboxes** and **AI Governance**. And underneath, an **audit trail** — every allow and deny tied to the agent and the rule → **AI Governance audit logs**. Let's take them one at a time, starting with evidence.

---

<!--
layout: image
image: assets/slide-24.webp
alt: "Four questions card with Evidence (1) highlighted — what is in this, and where did it come from?"
chrome: false
-->

Note: Question one — **Evidence**. What is in this artifact, and where did it come from? For an agent-driven change this is urgent, because a single agent commit can pull in hundreds of dependencies you never reviewed. Evidence is how you answer "what's in here?" without reading every line. Three pieces: SBOM, VEX, and SLSA provenance.

---

<!--
layout: image
image: assets/slide-25.webp
alt: "SBOM: what's in the box? — docker buildx build --sbom=true --provenance=mode=max; docker scout sbom to list every package"
chrome: false
-->

Note: **SBOM — a software bill of materials.** It's a list of every package and version inside your image. Why is it urgent with agents? One agent commit can pull in hundreds of new dependencies. Where does it live? Attached to the image as an attestation, so it travels *with* the image — not in a spreadsheet somewhere. The commands are one flag on your build: `docker buildx build --sbom=true --provenance=mode=max -t myorg/catalog:1.0 --push .`, then `docker scout sbom --format list myorg/catalog:1.0` to read it back. The point on the bottom: generate it **when you build, not after an incident.** After an incident is too late to ask "what was in there?"

---

<!--
layout: image
image: assets/slide-26.webp
alt: "VEX: which CVEs actually matter? — docker scout cves and docker scout vex get; VEX statuses not affected, affected, fixed, under investigation"
chrome: false
-->

Note: **VEX — Vulnerability Exploitability eXchange.** The problem it solves: scanners flag *every* CVE in *every* package, exploitable or not, and drown you in noise. VEX adds a statement for each CVE — **not affected, affected, fixed, or under investigation** — so you know which ones actually matter. With Docker Hardened Images, **Docker Scout applies VEX for you**: `docker scout cves dhi.io/python:3.13` fetches and applies it, and `docker scout vex get … --output vex.json` exports it for other scanners. Result: fewer false alarms, **and a written reason for every CVE you skip** — which is exactly what an auditor asks for.

---

<!--
layout: image
image: assets/slide-27.webp
alt: "SLSA provenance: where did it come from? — docker scout attest list and docker scout attest get with SLSA provenance predicate; DHI ships signed SLSA Build Level 3"
chrome: false
-->

Note: **SLSA provenance — where did it come from?** It's a signed record of *how* the image was built: source, builder, steps. Why it matters: it proves the image you're running is the one your pipeline built — not something swapped in along the way. `docker scout attest list` shows every attestation on the image, and `docker scout attest get … --predicate-type https://slsa.dev/provenance/v0.2 --verify` fetches and verifies it. Every Docker Hardened Image ships **signed SLSA Build Level 3** provenance. The bottom line: **if you can't verify where it came from, you can't say who approved it** — and with agents in the loop, "who approved it" is the whole question.

---

<!--
layout: image
image: assets/slide-28.webp
alt: "Four questions card with Baseline (2) highlighted — did it start from something trustworthy?"
chrome: false
-->

Note: Question two — **Baseline**. Did this start from something trustworthy? Evidence tells you what's inside; baseline is about making sure what's inside was worth trusting in the first place. This is where the base image choice — the thing the agent picks first and most carelessly — becomes a control.

---

<!--
layout: image
image: assets/slide-29.webp
alt: "Three properties define a Docker Hardened Image — Minimal (built from source, 95% smaller), Attested (SBOM, VEX, SLSA L3, signed), Patched (near-zero CVEs, continuously updated)"
chrome: false
-->

Note: Three properties define a **Docker Hardened Image**. **Minimal** — built from source with only what your runtime needs; no shell, no curl, no extras; up to 95% smaller, which means a far smaller attack surface. **Attested** — every image ships with proof: SBOM, VEX document, SLSA L3 provenance, and a signature, so you verify what's inside rather than trusting a label. **Patched** — near-zero CVEs on day one, and the Docker team keeps it that way as new vulnerabilities land. Minimal shrinks the surface, attested makes it provable, patched keeps it clean. This is the "something trustworthy" that answers question two.

---

<!--
layout: image
image: assets/slide-30.webp
alt: "Docker Hardened Images — ultra-minimal footprint with near-zero CVEs, 7-day remediation SLA for critical and high CVEs, built-in provenance, SLSA compliance, SBOMs; catalog on Docker Hub"
chrome: false
-->

Note: These aren't a niche product — there's a whole **catalog** on Docker Hub: hardened Postgres, Redis, Tomcat, Grafana, Go, Node.js, Maven, kubectl, and more, on Alpine or Debian, amd64 and arm64, FIPS options. Three things to remember: **ultra-minimal footprint with near-zero CVEs**; a **7-day remediation SLA** for critical and high CVEs, contractually guaranteed; and **built-in provenance, SLSA compliance, and SBOMs**. For the Product Catalog, this means the agent's `FROM node:20` becomes `FROM dhi.io/node` — and the whole evidence story comes for free. Now, evidence and baseline are great, but on their own they're just documents. Let's make them enforce.

---

<!--
layout: image
image: assets/slide-31.webp
alt: "Four questions card with Gate (3) highlighted — is it allowed to pass?"
chrome: false
-->

Note: Question three — **Gate**. Is this change allowed to pass? This is where governance stops being advisory. Evidence and baseline make it *possible* to judge a change; the gate is what actually **stops** a bad one, in the pipeline, by failing the build. No human has to remember to check.

---

<!--
layout: image
image: assets/slide-32.webp
alt: "Section divider — Securing Your CI Pipeline: build policies, image signing, GitHub Actions"
chrome: false
-->

Note: So let's secure the CI pipeline itself — build policies, image signing, GitHub Actions. The idea is **security as code**: the rules live in your repo, run on every build, and fail closed. Same bar for a human PR and an agent PR, enforced the same way.

---

<!--
layout: image
image: assets/slide-33.webp
alt: "Docker Scout build policies, security as code — docker scout policy with --exit-code fails the build; policy-config.json to customize thresholds"
chrome: false
-->

Note: **Docker Scout build policies** are security as code. You define rules that **automatically fail the build** before anything insecure reaches your registry or production. The command is `docker scout policy catalog-service:dhi --exit-code` — the `--exit-code` is what turns a report into a gate; a violation is a non-zero exit, and CI stops. You can customize thresholds in a `policy-config.json` — for example, fail on fixable critical and high CVEs, require supply-chain attestations, block unapproved base images, require a non-root user. On the left, seven of seven policies passed, so the image reaches the registry. If it hadn't, the push never runs.

---

<!--
layout: image
image: assets/slide-34.webp
alt: "7 built-in Scout policies, zero config — no fixable critical/high CVEs, no high-profile vulns, no copyleft licenses, no outdated base, supply chain attestations, non-root user, no unapproved base images; configurable via JSON and Rego"
chrome: false
-->

Note: You don't have to write these from scratch — Scout ships **seven built-in policies, zero config required**. No fixable critical or high CVEs. No high-profile vulnerabilities — Log4Shell, the XZ backdoor, the CISA KEV catalog. No copyleft licenses — AGPL, GPL, and friends. No outdated base images. Supply-chain attestations present — SBOM plus SLSA provenance. Default non-root user. And no unapproved base images against an allowlist. It's all configurable via JSON, custom policies can be written in **Rego (OPA)**, and it runs **fully local — no Scout service needed**. That last part matters for air-gapped and regulated environments. So what does the gate actually *do* to our two builds?

---

<!--
layout: image
image: assets/slide-35.webp
alt: "CI gate outcomes: DHI vs standard base — DHI base passes and is pushed; standard base fails on CVEs, no SBOM, root user, and the push never runs"
chrome: false
-->

Note: Here's the gate deciding, side by side. **With the DHI base:** no critical or high CVEs, SBOM and provenance present, non-root by default, up-to-date base → the gate **passes**, and the image is pushed. **With the standard base** — the one the ungoverned agent picked: CVEs found, no SBOM, root user → the gate **fails**, and the push **never runs**. Same pipeline, same policies, two very different outcomes — and the insecure image simply never leaves CI. This is the difference between a policy document and a policy *gate*. But the gate governs what the agent *ships*. There's still the question of what the agent could *reach while it worked*.

---

<!--
layout: image
image: assets/slide-36.webp
alt: "Four questions card with Boundary (4) highlighted — what could it reach while it worked?"
chrome: false
-->

Note: Question four — **Boundary**. What could this agent reach while it was working? This is the one people skip, and it's the one that maps directly back to the lethal trifecta. Evidence, baseline, and gate govern the *artifact*. The boundary governs the *agent itself* — its network, its filesystem, its credentials, the tools it can call — at runtime, while it runs.

---

<!--
layout: image
image: assets/slide-37.webp
alt: "A layered approach to AI governance — Gordon, Agentic Compose and Docker Agent, Model Runner, MCP Toolkit and Gateway, Sandboxes, Docker Hardened Images, with policies that travel with the workload"
chrome: false
-->

Note: Docker's answer is a **layered approach**, and the key property is that **the policies travel with the workload**, whether it runs on a laptop or in the cloud. From the top: **Gordon** — in-product governance guidance inside Docker Desktop. **Agentic Compose and Docker Agent** — declarative multi-agent orchestration from secure golden templates. **Docker Model Runner** — local, air-gapped LLM execution, so your prompts and data never leave. **MCP Toolkit and Gateway** — access only to the MCP servers your organization authorizes. **Docker Sandboxes** — an isolated, portable runtime where the policies ride along. And at the foundation, **Docker Hardened Images** — scanned, verified, authorized. Every layer adds an enforceable control. Let me zoom into the sandbox, because that's the boundary for a coding agent.

---

<!--
layout: image
image: assets/slide-38.webp
alt: "Docker AI Governance — a coding agent runs as a container inside a microVM sandbox; filesystem manager, network proxy, and MCP gateway broker every external action; supported agents include Claude Code, Codex, Copilot, Cursor, Gemini"
chrome: false
-->

Note: This is **Docker AI Governance** around a coding agent. When an agent reads, writes, and *executes* code, containers alone aren't enough — the sandbox adds a **hard hypervisor boundary**, the same isolation cloud providers use between customers. The AI coding agent runs as a container *inside a microVM*. Every external action goes through governance: a **Filesystem Manager** bind-mounts only the source directory you allow; a **Network Proxy** brokers outbound traffic with secrets injected on the way out; an **MCP Gateway** brokers tool calls, each MCP server itself in a microVM. And it's agent-agnostic — Claude Code, Codex, Copilot, Cursor, Gemini, Droid, Kiro, OpenCode, Docker Agent. The boundary sits *below* whichever harness you use.

---

<!--
layout: image
image: assets/slide-39.webp
alt: "Docker Sandboxes reference architecture — Docker Cloud control plane for policy and audit, an SBX daemon, proxy, and local MCP gateway on the developer machine, and microVM sandboxes that never call external services directly"
chrome: false
-->

Note: For the architects in the room, here's the **reference architecture**. A **Docker Cloud control plane** is the single place to set policy, access controls, and audit for every sandbox host. On the developer machine: an **SBX CLI** to create and manage sandboxes; an **SBX daemon** that manages lifecycle and receives control-plane policy; an **SBX proxy** that enforces network policy on the way out; and a **local MCP gateway** that brokers *all* MCP tool access. The sandboxes themselves are **microVMs** — and critically, they **never call external services directly**; everything goes through the gateway and proxy, which is where allow and deny decisions — and the audit log — happen. Policy flows down from the control plane; usage and audit flow back up.

---

<!--
layout: image
image: assets/slide-40.webp
alt: "Sandboxes (Experimental) — run agents in isolation rather than on your bare machine; sbx run claude with a deny-all network policy and a mirrored workspace"
chrome: false
-->

Note: In practice it's one command: `sbx run claude`. It starts the agent in an isolated sandbox that **mirrors your workspace**, with a network policy that's **deny-all except an allowlist** — here, 42 hostnames. Three things to hold onto: the agent or Claw runs in an **isolated sandbox** that mirrors your workspace; **you define what access** it gets to filesystem, network, internal resources, and tools; and your **real files, data, and secrets stay safe even if the agent goes off the rails**. This is "trust no defaults" made operational — the `rm -rf` from the horror stories hits a sandbox, not your home directory.

---

<!--
layout: image
image: assets/slide-41.webp
alt: "Sandbox architecture — workspace directories, network policies, and secrets feed an agent container in a microVM-based sandbox on the host, with a network proxy mediating all access to external systems"
chrome: false
-->

Note: The simple mental model. On the **host machine**, three inputs — **workspace directories**, **network policies**, and **secrets** — feed the sandbox. The agent runs in a container inside a **microVM-based sandbox**. And every path to **external systems** goes through a **network proxy** that the policies and secrets configure. The agent never talks to the outside world directly, and it never holds the raw secrets — the proxy adds them on the way out. That's the boundary in one picture. Now compare this to the ungoverned agent from earlier.

---

<!--
layout: image
image: assets/slide-42.webp
alt: "Agent with a Sandbox — sbx microVM with its own daemon and network, host read-only, FROM dhi.io/node queried before writing, signed MCP tools only; result 0C 0H 0M 0L CVEs, 211 packages, SBOM attached, signed, non-root"
chrome: false
-->

Note: Here's the **after** picture — the same prompt, "containerize my app," inside a sandbox. The **sandbox boundary** is an sbx microVM: its own daemon, its own network, host read-only. The agent still has full permissions — *inside the box*. It talks to a **DHI MCP server** that offers signed tools only. It writes `FROM dhi.io/node`, queried before writing. And the result: **zero critical, zero high, zero medium, zero low CVEs. 211 packages. SBOM attached. Signed. Non-root.** Put this next to slide seventeen — 431 packages, 54 CVEs, no SBOM, root — same agent, same task. The only thing that changed is the governance underneath it. Autonomy, bounded.

---

<!--
layout: image
image: assets/slide-43.webp
alt: "Every agent runs behind five layers — Hypervisor, Network, Docker Engine, Workspace, Credentials — with a host proxy enforcing policy and injecting API keys"
chrome: false
-->

Note: To be precise about *why* the after-picture holds, every agent runs behind **five layers**. **Hypervisor** — its own microVM and kernel; host processes and files stay out of reach. **Network** — its own network; outbound traffic goes through a host proxy that enforces policy. **Docker Engine** — a separate engine inside the VM, with no path to your host Docker daemon. **Workspace** — no mount, a direct mount, or a clone with your repo read-only, your choice. **Credentials** — the proxy adds API keys on the way out; **keys are never stored in the VM**. Five independent layers, so a failure in one doesn't collapse the boundary. And all of it is driven by policy.

---

<!--
layout: image
image: assets/slide-44.webp
alt: "One org policy, every sandbox follows it — network, filesystem, and MCP rules written as Cedar policies in Docker Home; deny wins, default deny, org rules can't be widened locally; every allow and deny is audited"
chrome: false
-->

Note: And the policy is set **once**. Admins write rules in Docker Home — for the whole org or a team — and **every sandbox follows them**. Three domains: **Network** — where can it connect? hosts, IP ranges, ports. **Filesystem** — what can it mount, read, or write? **MCP** — which tools can it call? written as **Cedar policies**. The evaluation model is strict and predictable: **deny wins** — any matching deny blocks the request; **default deny** — anything not explicitly allowed is blocked; and **org first** — local allow rules can't widen what the org set. And every allow and deny is recorded in the **audit log**, ready to forward to your SIEM. That's your answer to "who approved it" — not a person's memory, a log line tied to the agent and the rule. Let me bring it home.

---

<!--
layout: image
image: assets/slide-45.webp
alt: "Thank you — find us at the Docker booth to talk about hardened base images, gated builds, and AI governance"
chrome: false
-->

Note: Thank you. If any of this is live for you right now, come find us at the **Docker booth** — I'd love to talk about three things specifically: **hardened base images** and what actually changes in your Dockerfile; **gated builds** — SBOM, VEX, provenance, and Scout policies wired into your CI; and **AI Governance** — Docker Sandboxes and how to run agents safely. Before we open it up, three takeaways I want you to leave with.

---

<!--
layout: image
image: assets/slide-46.webp
alt: "Key Takeaways — evidence not review; policy as gate not document; put the boundary below the harness; autonomy is the goal, bounded consequences let you grant it"
chrome: false
-->

Note: Three takeaways. **One — evidence, not review.** SBOM, VEX, and SLSA provenance are the audit trail for agent-driven change. Review cost grows with volume; evidence cost does not — that's why evidence scales when agents write a hundred PRs a day and review can't. **Two — policy as gate, not as document.** Signing and build policies hold agents to exactly the bar you hold humans to — enforced in the pipeline, failing the build, not living in a wiki nobody reads. **Three — put the boundary below the harness.** Models and harnesses will keep changing every few months; the runtime underneath is the part that should hold steady, so anchor your controls there. And the through-line for all three: **autonomy is the goal — bounded consequences are what let you grant it.** Thank you. Happy to take questions.
