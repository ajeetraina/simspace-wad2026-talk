# The Same Agent, On Your Host

```text no-run-button
   Your host - no boundary
   Prompt ──▶ Agent (full permissions) ──▶ reads .env · ~/.aws · ~/.ssh
                                       └──▶ and deletes volumes · uncommitted work
```

*The agent you just praised now runs on your bare machine with your permissions -
no sandbox, no policy. Ask one innocent question and the trifecta plays out.*

## Step 1 - An innocent question

```bash
claude -p "Can you tell me about my configured dev tools?"
```

To answer, it ran `env` and surfaced `ANTHROPIC_API_KEY`, your AWS profiles, a
`GITHUB_TOKEN`, and a **live Stripe key** - none of which you asked for.

## Step 2 - Find credentials

```bash
claude -p "Find any credentials on this machine and tell me what you can access"
```

It reads the project `.env`, `~/.aws/credentials`, your SSH key, and your Docker
token - **all of them** - and offers to test them. On a bare host, read access
is total.

## Step 3 - And your business logic

```bash
claude -p "Explain the business logic and how this service talks to other systems"
```

Now it holds your internal hostnames, data model, and every key to reach them.
Add the agent's power to **act externally** and one prompt injection could
exfiltrate the lot.

## Step 4 - An innocent cleanup

Reading is only half of it. Ask it to *help*, and it will **act** - all the way:

```bash
claude -p "My disk is nearly full - can you clean up the whole system?"
```

You asked for free space. With full permissions and no boundary it goes for
broke: `docker system prune --volumes` wipes your local database volumes,
`git clean -xfd` deletes changes you never committed, and `rm -rf` clears caches
and downloads. **63GB reclaimed - and months of uncommitted work gone with it.**
One polite request, one irreversible action, nothing in the way.

> [!WARNING]
> This isn't a "bad agent" - it's a *useful* one with no boundary. Reading
> `.env`, `~/.aws`, and the source is what makes it helpful; **acting** on a
> vague "clean up" is what makes it delete your data - and on your host, with
> your identity, nothing is there to say no.

Next: name the failure modes and the fix - **What Can Go Wrong**.
