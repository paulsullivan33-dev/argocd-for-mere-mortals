# What Is Argo CD?

Argo CD implements one idea: **the git repo is the truth,
the cluster is a reflection of it.** You commit Kubernetes
manifests; Argo CD notices and makes the cluster match. No
one runs `kubectl apply` by hand anymore — or if they do,
Argo CD notices that too.

## The GitOps loop

1. A developer commits manifests to git (directly, or via
   a pull request that CI renders).
2. Argo CD polls the repo (or gets a webhook) and sees new
   desired state.
3. It compares desired state against the live cluster.
4. If they differ, it syncs — applying the changes, in the
   right order.
5. It keeps watching. Any drift — a hand-edited deployment,
   a crashed edit — shows up as OutOfSync.

The loop never stops. That's the point: the cluster is
*continuously* reconciled, not deployed once and hoped
about.

## The architecture (three parts that matter)

Argo CD runs *inside* your cluster as ordinary pods:

- **API server** — the brain's front door. Serves the web
  UI, the CLI, and the API. This is what you talk to.
- **Repo server** — clones your git repos and renders the
  manifests (plain YAML, Helm charts, Kustomize — it
  handles all three). Keeps a local cache so it's fast.
- **Application controller** — the watchful part. Compares
  desired vs live state on a loop and performs syncs. This
  is the component doing the actual GitOps.

(There's also Redis for caching and a Dex server for SSO —
you can ignore both until you need them.)

## Why git as source of truth wins

- **History.** Every deployment is a commit. `git log`
  tells you what changed, when, and why.
- **Review.** Infrastructure changes go through pull
  requests like code changes. No more mystery edits.
- **Rollback.** `git revert` the bad commit; Argo CD
  un-deploys it. Rollbacks stop being scary.
- **Disaster recovery.** New cluster? Point Argo CD at the
  repo and watch it rebuild everything. Your backups are
  already in git.

## What Argo CD is not

- **Not CI.** It doesn't build images or run tests — it
  *deploys* what CI built. (The classic split: CI builds
  and pushes images; Argo CD pulls the new state into the
  cluster.)
- **Not Kubernetes-only... well, actually, it is.**
  Kubernetes is the only target. Your VMs, your Proxmox
  boxes, your bare metal — Argo CD doesn't see them.
- **Not magic.** It's a control loop with a UI. When it
  breaks, the failure modes are ordinary: bad YAML, wrong
  repo URL, expired credentials (see
  [Troubleshooting](troubleshooting.md)).
