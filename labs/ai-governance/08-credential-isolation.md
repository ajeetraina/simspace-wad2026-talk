# Lab 3: Credential Isolation

```text no-run-button
   MicroVM (sandbox)                     Host - sbx proxy                Service
   ANTHROPIC_API_KEY = proxy-managed ─▶  1 match destination → service   api.anthropic.com
   (sentinel only)                       2 read real key (keychain)      (sees real key)
                                         3 inject Authorization header ─────────▶
```

*The agent authenticates without ever holding the key: it sees only the
`proxy-managed` sentinel, while the proxy reads the real secret from the host and
injects it per request.*

> **In plain terms:** the agent **uses** your API key without ever **seeing** it.
> The real secret stays on the host; the proxy adds it to each request on the way
> out. Even a hijacked agent has nothing to steal.

Back to the **Product Catalog**. To containerise and run it, the coding agent needs
**real credentials**: its own **Anthropic key** to think, plus the app's own secrets -
the **AWS keys** for the product-image S3 bucket and the token for the **Inventory
service** the catalog calls. You don't want the agent *reading* secrets off disk - but
these it legitimately needs to *use*. So:

> *If the agent can't read my keys, how does it authenticate to the services the
> catalog actually depends on?*

The answer is **credential isolation**: the real secret never enters the sandbox.
A host-side proxy injects it per request; the sandbox sees only a sentinel.

> [!NOTE]
> This is a **different control** from network/filesystem rules. Those are
> admin-governed org policies (`ORIGIN: remote`). Credential isolation is a
> sandbox runtime protection you configure **developer-side** with `sbx secret`,
> the OS keychain, or OAuth. There's no Admin Console toggle for it.

## Step 1 - Store the key on the host, before you run the agent

Set this up **first** - the proxy can only inject a secret that already lives on the
host. Store the catalog agent's Anthropic key in the OS keychain (encrypted at rest,
and keychain secrets take precedence over any env var):

```bash
sbx secret set -g anthropic
```

```bash
sbx secret ls
```

The key now lives on the **host** - not in your shell history, and not in any
sandbox. With it in place, the proxy has something to inject; let's launch the
catalog agent and confirm it never sees it.

## Step 2 - Launch the catalog sandbox and look for the key

Open a sandbox on the catalog workspace - the same box the agent uses to build and
run the Product Catalog:

```bash
sbx run shell ~/workdemo/creds
```

Look at the credential the coding agent would use:

```bash
echo "ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY"
```

The variable exists - tools expect it - but its value is `proxy-managed`, not the
key you just stored. There is no live secret anywhere in the sandbox.

## Step 3 - Watch the proxy inject the real credential

Still inside, make the call the coding agent makes to think:

```bash
curl https://api.anthropic.com -sS -o /dev/null -w "anthropic: %{http_code}\n"
```

It reaches Anthropic (404). Now exit and inspect the proxy log on the **host**:

```bash
exit
```

```bash
sbx policy log
```

The request to `api.anthropic.com` is logged as `forward` (allowed). The proxy
matched the destination to the `anthropic` service, read the real key you stored in
Step 1, and injected the `Authorization` header - all without the key ever touching
the sandbox.

## Step 4 - The catalog's own secrets (custom)

The Product Catalog also calls an internal **Inventory service**. Declare a **custom
secret** keyed to its host and env var, so the app can reach it without the agent
ever holding the token:

```bash
sbx secret set-custom -g --host api.inventory.internal --env INVENTORY_API_KEY
```

Inside the sandbox, `INVENTORY_API_KEY` shows a placeholder; the proxy substitutes
the real value on requests to `api.inventory.internal`. The same pattern covers the
AWS keys for the product-image S3 bucket - stored on the host, injected on the wire.

## Read the results

| What | Where the secret lives | What the sandbox sees |
| --- | --- | --- |
| `ANTHROPIC_API_KEY` (agent) | Host keychain (Step 1) | `proxy-managed` sentinel |
| Allowed API call | Injected by proxy on the wire | Never the raw value |
| Inventory / S3 secret | Host, keyed to host + env | Placeholder; substituted per request |
| SSH key | Host SSH agent | Can sign, can't read the key |

## Credentials, contained

Even for the calls the catalog agent is *supposed* to make, a prompt injection can't
exfiltrate a usable key - because there is no usable key inside the box.

That closes the **credential** boundary: the agent uses your keys without ever seeing
them. Next, contain what the agent can *reach* on the wire - **Network Enforcement**.
