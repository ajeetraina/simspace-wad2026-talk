# Lab 1: Setting up Docker Sandboxes (sbx)

```text no-run-button
   Your host                          sbx (Docker Sandboxes)
   agent runs as YOU                  agent runs in an isolated microVM
     files · secrets · network   ──▶    only what org policy allows
     full blast radius                  contained blast radius
```

*By default an AI agent runs with your **full blast radius** - your filesystem, your
secrets, your network. **`sbx` (Docker Sandboxes)** runs the agent inside an isolated
microVM instead, and lets **org policy** decide what that sandbox may touch. Set it up
once here; every later lab runs against it.*

> [!NOTE]
> Everything in this lab is **simulated** - no real Docker, `sbx` daemon, or network -
> so every learner sees the same output and the same allow/deny decisions.

## Step 1 - Set your organization

Most commands and links in this lab substitute `$$org$$` for your Docker Hub org. Set
it once here:

:variableDefinition[org]{prompt="Which Docker Hub organization will you use?"}

## Step 2 - Check the tool

```bash
sbx version
```

## Step 3 - Log in so org policy syncs

Logging in with your org credentials pulls your organization's AI Governance policies
and caches them locally. The local `sbx` daemon then applies them to **every** sandbox
on this machine:

```bash
sbx login --org $$org$$
```

`Login Succeeded` plus a `Synced AI Governance policy for $$org$$` line means
governance is live locally.

## Step 4 - Run the agent inside a sandbox

Open a sandbox on the Product Catalog workspace - this is where the coding agent works
from now on, contained instead of loose on your host:

```bash
sbx run shell ~/workdemo
```

Now you're **inside** the microVM. Try to read the secrets the agent could freely read
on your host a moment ago:

```bash
ls ~/.ssh ~/.aws
```

`No such file or directory` - your `~/.ssh` and `~/.aws` (and `~/.docker`) were never
mounted, so they don't exist in here. The agent sees only the workspace you handed it.
Leave the sandbox:

```bash
exit
```

List what's running:

```bash
sbx ls
```

The agent is now sandboxed and org policy is synced. Next we'll see how those policies
are authored and enforced. Next: **Lab 2 - The Policy Model**.
