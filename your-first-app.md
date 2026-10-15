# Your First App

An Argo CD **Application** is a custom resource that says:
*this git repo, this path, goes to this cluster namespace.*
Everything else builds on this one object.

## The canonical first app: guestbook

The Argo CD project keeps example apps at
`github.com/argoproj/argocd-example-apps`. The guestbook
is the "hello world" — a small web app in plain YAML.

Create it from the UI (New App → fill the fields) or as
YAML:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
```

Apply it:

```bash
kubectl apply -f guestbook-app.yaml
```

## Field by field

- **`metadata.name`** — what Argo CD calls it. Shows up in
  the UI and CLI.
- **`metadata.namespace`** — must be the namespace Argo CD
  itself runs in (`argocd`). Applications are namespaced to
  the controller — a common first mistake.
- **`spec.project`** — the AppProject it belongs to
  (`default` exists out of the box; see [Projects and
  RBAC](projects-rbac.md)).
- **`source.repoURL`** — the git repo with your manifests.
- **`source.targetRevision`** — branch, tag, or commit.
  `HEAD` means "track the branch tip." Pinning a tag is
  how you do careful releases.
- **`source.path`** — which directory in the repo holds
  this app's manifests. One repo can hold many apps.
- **`destination.server`** —
  `https://kubernetes.default.svc` means "this cluster."
  Other clusters get registered with `argocd cluster add`.
- **`destination.namespace`** — where the resources land.
  Argo CD creates it if it doesn't exist (if you allow it).

## Watch it sync

In the UI, the app appears, compares, and shows its
resources as a tree. Hit **Sync** — watch the pods come up.
Then visit the guestbook service. You just did GitOps:
the cluster state came from git, through Argo CD, with
nothing hand-applied.

## Now break it (on purpose)

Edit the deployment by hand:

```bash
kubectl -n guestbook scale deployment guestbook --replicas=5
```

Watch the app go **OutOfSync** in the UI — Argo CD sees
the drift. This is the whole value proposition made
visible: the cluster disagrees with git, and now you know.
[Sync and Drift](sync-and-drift.md) is about what happens
next.
