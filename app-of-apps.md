# App of Apps

The **app-of-apps pattern**: one parent Application whose
"manifests" are *other Applications*. Sync the parent, and
it creates the children, which sync themselves. A whole
cluster bootstrapped from a single `kubectl apply`.

## Why it exists

Without it, every app needs its own `kubectl apply` (or
UI click) to register with Argo CD — which is exactly the
hand-driven step GitOps was supposed to eliminate. The
parent app closes the loop: the *set of applications* is
itself declared in git.

## How it looks

A repo layout like:

```
apps/
  parent.yaml          # the app-of-apps Application
  guestbook.yaml       # Application: guestbook
  monitoring.yaml      # Application: monitoring
  ingress.yaml         # Application: ingress-nginx
```

And the parent:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/you/infra.git
    targetRevision: HEAD
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
```

The parent's `path: apps` contains Application manifests —
which Argo CD applies to the cluster, which registers the
children, which then sync their own workloads. One manual
step, ever:

```bash
kubectl apply -f apps/parent.yaml
```

## The gotchas

- **The parent syncs the children, not the workloads.**
  The parent going green means the *Applications* exist —
  each child still syncs on its own policy.
- **Don't let children fight the parent.** If a child
  Application's manifest is also managed by something
  else, they'll argue forever. One owner per object.
- **Bootstrapping secrets.** The repo holds manifests, not
  secrets — credentials for the apps come from somewhere
  else (sealed secrets, external secrets operator, or a
  manual step you document loudly). App-of-apps doesn't
  solve secret management; nothing in git should try to.
- **Blast radius.** One parent syncing everything means
  one bad commit can touch everything. This is why the
  parent usually stays on *manual* sync even when the
  children are automated.

## When you've outgrown it

App-of-apps scales to dozens of apps comfortably. Past
that, teams move to **ApplicationSets** — Argo CD's
built-in generator that creates Applications from
templates (one per cluster, per environment, per team).
Same idea, less YAML copying. Learn app-of-apps first;
ApplicationSets will make immediate sense afterward.
