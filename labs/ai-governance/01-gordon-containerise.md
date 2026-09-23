# Gordon Containerises the Product Catalog

```text no-run-button
   You ── "containerise this app" ──▶  Gordon (docker ai)  ──▶  product-catalog:latest
                                          │  best practices:
                                          │  multi-stage · npm ci
                                          │  non-root · .dockerignore
                                          ▼
                                 a clean, working image
                                 …still on the fat node:20 base
```

*Hand a real app to Docker's built-in agent and it containerises it **well** -
multi-stage, non-root, a proper `.dockerignore`. But "best practices" can't fix the
base image it chose: that's what the next two sections tackle.*

Let's start where real work starts: a real service and an agent doing a real task.
The [Product Catalog service](https://github.com/ajeetraina/product-catalog-demo-showcase)
is a Node.js + Express API backed by Postgres, S3, and Kafka - with **no Dockerfile
yet**. You'll hand it to **Gordon**, the AI agent built into the Docker CLI
(`docker ai`), and ask it to containerise the app.

## Step 1 - Meet the app

```bash
git clone https://github.com/ajeetraina/product-catalog-demo-showcase
```

Take a look at what the agent will work on:
:filelink[package.json]{path="product-catalog-demo-showcase/package.json"}

Get the lay of the land before you hand it off. Note there's **no `Dockerfile`
and no Compose file** yet - this repo ships source only:

```bash
tree product-catalog-demo-showcase
```

## Step 2 - Ask Gordon to containerise it

Gordon is Docker's agent - it reads your project, follows containerisation best
practices, and can build the image for you. Ask it to do exactly that:

```bash
docker ai "Containerise the product catalog for production following best practices"
```

Gordon reads the project, writes a **multi-stage** `Dockerfile` (build stage +
minimal runtime), runs `npm ci`, adds a **non-root** user and a `.dockerignore`,
writes a `compose.yaml`, and builds `product-catalog:latest`. Run `tree` again to
see what it added to the once source-only repo:

```bash
tree product-catalog-demo-showcase
```

Review what Gordon wrote - this is a genuinely well-formed Dockerfile:

```bash
cat product-catalog-demo-showcase/Dockerfile
```

```bash
cat product-catalog-demo-showcase/compose.yaml
```

## Step 3 - Best practices ≠ a clean image

Gordon did a good job. The build is multi-stage, it runs as an unprivileged user,
it copies only what it needs. And yet - it still starts from the convenient, **fat
`node:20` base**. Best-practice *layering* can't undo what the *base* drags in.

> [!NOTE]
> This is the crucial point: an agent applying every containerisation best practice
> still ships whatever CVEs live in the base image it picked. You can't lint your way
> out of a vulnerable base - you have to **measure it**, then **swap it**.

That's exactly the next two sections:

| Next | What it does |
| --- | --- |
| **Docker Scout** | Scan `product-catalog:latest` and see what the base dragged in |
| **Hardened Images (DHI)** | Swap the base, re-scan, watch the CVEs collapse |

First, let's see the damage. Head to **Docker Scout: What's Inside**.
