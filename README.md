# Argo CD for Mere Mortals

**Argo CD** is a GitOps delivery tool for Kubernetes: you
describe what should be running in a git repo, and Argo CD
keeps your cluster matching it. This guide covers what it
is, installing it, deploying your first app, how sync and
drift work, the app-of-apps bootstrap pattern, projects and
RBAC basics, and fixing the things that break.

No Kubernetes wizardry assumed beyond the basics — if
you've run `kubectl apply` and survived, you're ready.

## What this guide covers

- [**What Argo CD is**](what-is-argocd.md) — the GitOps
  loop, the architecture, and why git as source of truth
- [**Installing**](installing.md) — Argo CD on a local
  cluster, the CLI, and your first login
- [**Your first app**](your-first-app.md) — the Application
  resource, field by field, deploying the guestbook
- [**Sync and drift**](sync-and-drift.md) — manual vs
  automated sync, prune, self-heal, and sync waves
- [**App of apps**](app-of-apps.md) — bootstrapping a whole
  cluster from one Application
- [**Projects and RBAC**](projects-rbac.md) — AppProjects
  and who may touch what
- [**Troubleshooting**](troubleshooting.md) — symptom → fix,
  seven ways

## What it doesn't cover

Argo Rollouts (progressive delivery — a separate project),
deep Helm/Kustomize authoring, and multi-cluster fleet
management at enterprise scale. This guide gets you
*running* Argo CD and understanding it — the advanced
topography comes later.

## How to use this guide

Commands were checked against current Argo CD documentation
before shipping (v3.x line, October 2026). **Check:**
blocks show verified commands and real output shapes.
Version numbers move; the concepts don't. Anything that
needs a cluster is marked — minikube is plenty.

## Chapters

1. [What Is Argo CD?](what-is-argocd.md)
2. [Installing](installing.md)
3. [Your First App](your-first-app.md)
4. [Sync and Drift](sync-and-drift.md)
5. [App of Apps](app-of-apps.md)
6. [Projects and RBAC](projects-rbac.md)
7. [Troubleshooting](troubleshooting.md)
8. [Glossary](glossary.md)

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [GitOps for Mere Mortals](https://paulsullivan33-dev.github.io/gitops-for-mere-mortals/)
- [Kubernetes for Mere Mortals](https://paulsullivan33-dev.github.io/kubernetes-for-mere-mortals/)
- [GitHub Actions for Mere Mortals](https://paulsullivan33-dev.github.io/github-actions-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/argocd-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/argocd-for-mere-mortals/blob/main/LICENSE-CODE).
