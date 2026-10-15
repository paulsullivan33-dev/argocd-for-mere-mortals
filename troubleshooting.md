# Troubleshooting

Symptom → fix, in the order you'll hit them.

## App stuck "Progressing" forever

**Symptom:** sync starts, resources show Progressing, never
become Healthy.

**Cause:** usually the workload itself — image pull
failing, CrashLoopBackOff, a service with no endpoints.

**Fix:** this isn't an Argo CD problem; it's Argo CD
faithfully reporting a broken app. `kubectl -n <ns> get
pods` and read the pod events. Fix the app; Argo CD will
notice.

## Everything is OutOfSync right after syncing

**Symptom:** you sync, it goes green, seconds later it's
OutOfSync again with no new commits.

**Cause:** the live object has fields the git manifests
don't — typically status fields, auto-generated values,
or defaults the API server injects.

**Fix:** tell Argo CD to ignore those fields with
`ignoreDifferences` on the Application:

```yaml
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

(`/spec/replicas` is the classic: an HPA manages
replicas, git says 3, they fight forever. Ignoring the
field ends the war.)

## "Permission denied" on sync

**Symptom:** sync fails; events mention RBAC or project
restrictions.

**Cause:** the app's repo or destination isn't allowed by
its AppProject, or the Argo CD service account lacks
cluster permissions.

**Fix:** check the project's `sourceRepos` and
`destinations` first — a typo'd repo URL is the usual
suspect. Then check whether the destination namespace
exists (or enable `CreateNamespace=true`).

## Can't reach private repos

**Symptom:** app shows "unable to clone" or
authentication errors.

**Cause:** Argo CD has no credentials for the repo.

**Fix:** UI → Settings → Repositories → Connect Repo, or
`argocd repo add` from the CLI. For private repos you
need a deploy key or token *before* the app will ever go
green — Argo CD can't prompt you mid-sync.

## Webhook never fires (polling feels slow)

**Symptom:** commits take ~3 minutes to show up.

**Cause:** that's the default poll interval — not a bug.

**Fix:** it resolves on its own, or configure a webhook
from your git host to Argo CD for near-instant updates.
Don't lower the poll interval to seconds; you'll hammer
your git host for nothing.

## Self-heal reverted my emergency fix

**Symptom:** a hand-applied hotfix during an incident
vanished minutes later.

**Cause:** `selfHeal: true` doing exactly what it was
told.

**Fix:** commit the hotfix to git (the right answer —
now it's reviewed and permanent), or temporarily pause
the app (`argocd app suspend`) while you fight the fire.
Afterward, decide whether that app should really be
self-healing — incident-prone apps sometimes shouldn't.

## The UI shows apps I deleted

**Symptom:** deleted Application CRs linger as ghosts.

**Cause:** the app-of-apps parent still lists them, or
finalizers are stuck.

**Fix:** remove them from the parent's repo first (the
parent will prune them if configured). For stuck
finalizers, `kubectl patch` the Application to remove
the finalizer — carefully, understanding you're
bypassing the safety catch.
