# Sign It, Then Gate It (CI)

```text no-run-button
   git push ──▶  secure-build workflow
                 ┌────────────────────────────────────────────┐
                 │ build → sbom → POLICY GATE → sign → push    │
                 │                    │                         │
                 │        no critical/high CVEs? ── no ──▶ FAIL │  (image not promoted)
                 │                    └─ yes ──▶ cosign sign ──▶ push │
                 └────────────────────────────────────────────┘
```

*A hardened image is only safe if a vulnerable one **can't** ship. This section adds
a CI pipeline that gates on a Docker Scout policy and **fails closed** - then signs
what passes with cosign, so production only ever runs verified artifacts.*

You hardened the image by hand. Now make it a **boundary**: a pipeline that refuses
to promote anything that doesn't meet policy. Open the **CI** tab (top-right) as you
work through this - the run appears there.

## Step 1 - Define the policy gate

The gate is three Docker Scout policies. Save them into the repo:

```yaml save-as=product-catalog-demo-showcase/.docker/scout-policy.yaml
version: "1"
policies:
  - name: no-critical-cves      # is it safe?
    type: vulnerability
    severity: critical
    action: fail
  - name: require-sbom          # what's in it?
    type: attestation
    attestation: sbom
    action: fail
  - name: require-provenance    # where did it come from?
    type: attestation
    attestation: slsa-provenance
    action: fail
```

## Step 2 - Add the signing key

Signing proves an image is the one your pipeline built. Generate a cosign key pair
(CI uses keyless Sigstore signing; this is the local equivalent):

```bash
cosign generate-key-pair
```

## Step 3 - Add the workflow

Save a workflow that builds, scans against the policy, signs, and pushes - the gate
step **fails closed** so an image with critical/high CVEs never reaches the registry:

```yaml save-as=product-catalog-demo-showcase/.github/workflows/secure-build.yaml
name: secure-build
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build (with SBOM + provenance)
        run: docker build --sbom=true --provenance=mode=max -t product-catalog:latest .
      - name: Policy gate (Docker Scout)
        uses: docker/scout-action@v1
        with:
          command: policy
          image: product-catalog:latest
          organization: $$org$$
          exit-on: policy          # fail the job if any policy fails
      - name: Sign (cosign, keyless)
        run: cosign sign --yes product-catalog:latest
      - name: Push
        run: docker push product-catalog:latest
```

## Step 4 - Check the gate locally first

Before pushing, evaluate the same policy locally. Because you already switched to a
hardened base, it passes:

```bash
docker scout policy product-catalog:latest
```

`PASSED (3 of 3)`. Had you run this on the **un-hardened** image from earlier, it
would read `FAILED - 2 critical, 12 high` and the pipeline would refuse to promote it.
That's the point: the gate is what makes "use a hardened base" enforceable.

## Step 5 - Ship it through the pipeline

Commit and push. The `secure-build` run fires - watch it in the **CI** tab:

```bash
git add .
```

```bash
git commit -m "Add secure-build pipeline (Scout gate + cosign signing)"
```

```bash
git push
```

The run goes green: **build → sbom → policy-gate → sign → push**. The image is
scanned, passes the gate, is signed by cosign, and is promoted - all automatically.

> [!NOTE]
> The gate is **state-derived**: it passes because the image is hardened. If you
> reset the lab and push *before* hardening, the `policy-gate` step fails and the
> run stops there. Fix the base, hit **Re-run jobs** in the CI panel, and the same
> pipeline goes green - the "fail → fix → re-run" loop, exactly like GitHub Actions.

## What you built

The supply-chain half is complete: Gordon **built** it, Scout **measured** it, DHI
**hardened** it, and CI now **gates and signs** it. Only verified, low-CVE, signed
images reach production.

Now the other half. A safe artifact is not a safe *agent* - Gordon ran with your
full access this whole time. Next we contain that blast radius, starting with
**The Policy Model**.
