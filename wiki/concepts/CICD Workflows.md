---
type: concept
title: "CI/CD Workflows"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - "CI/CD"
  - "GitHub Actions"
  - "GitLab CI"
  - "Self-Hosted Runners"
tags:
  - cicd
  - github-actions
  - gitlab
  - automation
  - devops
status: developing
related:
  - "[[Docker]]"
  - "[[Ansible]]"
  - "[[Terraform]]"
  - "[[Cargo Workflow]]"
---

# CI/CD Workflows

**CI** (continuous integration) runs build/test/lint on every push so breakage is
caught early; **CD** (continuous delivery/deployment) takes a passing build and
ships it (to a registry, a server, a static host). Both are expressed as YAML
pipelines living in the repo, triggered by git events and run on **runners**.

> [!key-insight]
> A pipeline is just "run these steps in a clean environment on every change."
> The portable core — keep logic in scripts (`make`, `just`, shell) the pipeline
> *calls*, not buried in YAML — means you can run the same checks locally and
> migrate between GitHub Actions and GitLab CI with mostly the trigger/syntax
> layer changing.

## Core concepts (platform-agnostic)

- **Trigger / event** — what starts a run (push, PR/MR, tag, schedule, manual).
- **Stage / job** — a unit of work in its own environment; jobs can run in
  parallel or depend on each other (DAG).
- **Step** — a command (or reusable action) inside a job.
- **Runner / executor** — the machine that runs the job (hosted or self-hosted).
- **Artifacts** — files passed between jobs or out of the pipeline (binaries,
  reports).
- **Cache** — reused between runs to speed up deps (npm/cargo/pip).
- **Secrets / variables** — injected credentials, never committed.
- **Matrix** — fan a job across versions/OSes (e.g. test on 3 Go versions).

## GitHub Actions

Workflows live in `.github/workflows/*.yml`. A workflow has jobs; jobs have steps;
steps run shell or a reusable **action** (`uses:`).

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest          # GitHub-hosted runner
    strategy:
      matrix:
        go: ["1.22", "1.23"]        # matrix fan-out
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: "${{ matrix.go }}" }
      - run: go test ./...
      - uses: actions/upload-artifact@v4
        with: { name: bin, path: ./dist }

  deploy:
    needs: test                     # runs only if test passed
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
        env:
          TOKEN: ${{ secrets.DEPLOY_TOKEN }}   # repo/org secret
```

- **Marketplace actions** (`uses: owner/action@vN`) — pin to a tag/SHA; treat
  third-party actions as supply-chain (pin SHAs for sensitive workflows).
- **Secrets** — repo/org/environment scoped (`${{ secrets.X }}`); **environments**
  add required reviewers/approvals for deploys.
- **Cache** — `actions/cache` or built-in setup-action caching.

### Self-hosted runners

When you need more power (GPU, RTX builds), private network access, or to avoid
hosted minutes, register your own runner:

```bash
# from repo/org Settings → Actions → Runners → New self-hosted runner
./config.sh --url https://github.com/<org>/<repo> --token <reg-token>
./run.sh                      # or install as a service:
sudo ./svc.sh install && sudo ./svc.sh start
```

Target it from a job with labels:

```yaml
jobs:
  build:
    runs-on: [self-hosted, linux, gpu]
```

> [!warning] **Never attach self-hosted runners to public repos.** A fork PR can
> run arbitrary code on your runner. Self-hosted runners are for private repos (or
> tightly controlled environments) only. Run jobs in ephemeral containers/VMs and
> rotate runners where possible.

> [!key-insight] Self-hosted runners are the bridge to homelab compute: a runner
> on a [[Proxmox]] VM (reachable over [[Tailscale]]) gives Actions access to local
> hardware and private services without exposing anything inbound. Ephemeral
> (`--ephemeral`) runners that deregister after one job are the safer pattern.

## GitLab CI

A single `.gitlab-ci.yml` at the repo root. Jobs are grouped into **stages** that
run in order; jobs in the same stage run in parallel.

```yaml
# .gitlab-ci.yml
stages: [test, build, deploy]

variables:
  GIT_DEPTH: "10"

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths: [.cargo/]

test:
  stage: test
  image: rust:latest             # job runs in this container
  script:
    - cargo test                 # see [[Cargo Workflow]]

build:
  stage: build
  image: rust:latest
  script: [cargo build --release]
  artifacts:
    paths: [target/release/app]

deploy:
  stage: deploy
  script: [./deploy.sh]
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'   # only on main
  environment: production
```

- **`image:`** — jobs run in a container (Docker executor); `services:` add
  sidecars (e.g. a Postgres for tests).
- **Predefined vars** — `CI_COMMIT_*`, `CI_PIPELINE_*`, etc.; **CI/CD variables**
  (masked/protected) hold secrets.
- **`rules:`/`only:`/`except:`** — conditional execution.
- **`needs:`** — DAG ordering across stages for faster pipelines.

### GitLab Runners

GitLab runs jobs on **runners** you register (GitLab also offers shared/SaaS
runners). Register with an **executor** — Docker is the common choice:

```bash
gitlab-runner register \
  --url https://gitlab.example \
  --registration-token <token> \
  --executor docker \
  --docker-image alpine:latest
```

Executors: `shell`, `docker`, `docker-autoscaler`, `kubernetes`. Tag jobs
(`tags: [gpu]`) to pin them to specific runners.

## GitHub Actions vs GitLab CI

| | GitHub Actions | GitLab CI |
|--|----------------|-----------|
| Config | many files in `.github/workflows/` | one `.gitlab-ci.yml` |
| Unit | workflow → jobs → steps | stages → jobs |
| Reuse | Marketplace actions | `include:`, `extends:`, templates |
| Runners | GitHub-hosted + self-hosted | shared + self-managed (executors) |
| Strength | huge action ecosystem | tightly integrated DevOps platform |

> [!note] Both deploy the same way in practice: build → test → package (often a
> [[Docker]] image) → push to a registry → release (rsync/ssh, k8s, or trigger
> [[Ansible]]/[[Terraform]]). The brain's own publish flow is a change-gated
> systemd timer rather than a CI runner — a deliberately minimal "CD".

## Pros / trade-offs

**Pros**
- Catches regressions on every change; enforces tests/lint/format as gates.
- Reproducible builds in clean environments; automated, auditable deploys.
- Matrix testing across versions/platforms with little effort.

**Trade-offs**
- YAML sprawl and platform lock-in if logic lives in the pipeline, not scripts.
- Self-hosted runners are real security/maintenance surface (isolation, patching).
- Caching/secrets misconfig are common footguns; slow pipelines erode the feedback
  loop they exist to provide.

## Related

- [[Docker]] — jobs run in containers; images are the usual build output
- [[Ansible]] · [[Terraform]] — common deploy/provision steps a pipeline calls
- [[Cargo Workflow]] — example of the build/test commands a CI job wraps
