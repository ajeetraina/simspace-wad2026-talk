# Supply Chain Security When Agents Write the Code

The **WeAreDevelopers 2026** talk by [Ajeet Singh Raina](https://github.com/ajeetraina),
Developer Advocate at Docker — presented as an in-browser
[Simspace](https://github.com/dockersamples/simspace) slide deck.

> Agents now write code at every stage of your pipeline. This deck lays out the
> four questions every agent-driven change must answer — **Evidence, Baseline,
> Gate, Boundary** — and the Docker stack that enforces them: Docker Scout
> (SBOM/VEX/SLSA), Docker Hardened Images, Scout build policies, and Docker
> Sandboxes + AI Governance.

The companion hands-on lab lives in
[`ajeetraina/simspace-ai-governance-demo`](https://github.com/ajeetraina/simspace-ai-governance-demo).

## What's here

```
labs/supply-chain-security/
  labspace.yaml     # deck manifest (kind: slides, brand, theme)
  deck.md           # 46 full-bleed slides + speaker notes (Note:)
  assets/           # slide-01..46.webp + Docker logos
```

Each slide is a full-bleed image exported from the master deck, so branding
matches exactly. The talk track for every slide lives inline as a `Note:` block —
press **`S`** in the presenter to open the speaker-notes view.

## Preview locally

You only need Docker.

```bash
docker compose up dev              # live preview at http://localhost:5173
docker compose run --rm validate   # validate the deck (fails on errors)
```

Edit `labs/supply-chain-security/*` and refresh the browser to see changes.

## Deploy (GitHub Pages)

Pushing to `main` runs [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml),
which validates and publishes to GitHub Pages via the reusable
`dockersamples/simspace` workflow.

**One-time setup:** repo **Settings → Pages → Source: "GitHub Actions"**.

## Rebuilding the slide images

The slides are rendered from the master PDF. To regenerate them:

```bash
PDF="Supply Chain Security When Agents Write the Code.pdf"
pdftoppm -png -scale-to-x 2000 -scale-to-y 1125 "$PDF" /tmp/slide
i=0; for f in $(ls /tmp/slide-*.png | sort); do
  i=$((i+1)); n=$(printf "%02d" $i)
  cwebp -q 88 "$f" -o "labs/supply-chain-security/assets/slide-$n.webp"
done
```

## The four questions

| Question | Control | Docker tool |
|----------|---------|-------------|
| **Evidence** — what's in it, where from? | SBOM, VEX, provenance on the image | `docker buildx`, Docker Scout |
| **Baseline** — did it start from something trustworthy? | Minimal, attested, patched base | Docker Hardened Images |
| **Gate** — is it allowed to pass? | Policy check before promotion, fail closed | Docker Scout policies |
| **Boundary** — what could it reach while it worked? | Isolated runtime, default-deny net/fs/tools | Docker Sandboxes, AI Governance |

*Evidence and baseline make governance possible. Gate and boundary make it real.*
