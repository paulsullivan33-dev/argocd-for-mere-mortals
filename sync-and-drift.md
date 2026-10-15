# Sync and Drift

**Sync** is Argo CD making the cluster match git.
**Drift** is the cluster disagreeing with git. This chapter
is the knobs that control both.

## Manual vs automated sync

By default, sync is **manual**: Argo CD shows you the
diff, you press Sync. Nothing changes until a human says
so. This is the right default while learning — and the
right choice for production in many teams.

**Automated** sync applies changes as soon as they're
detected:

```yaml
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Two sub-options, both worth understanding separately:

- **`prune: true`** — delete resources that were removed
  from git. Without it, deleting a manifest from the repo
  leaves the resource running forever. Prune is how "git
  is the truth" stays true in the *removal* direction.
- **`selfHeal: true`** — revert hand-edits back to git
  state automatically. That `kubectl scale` experiment
  from the last chapter? With self-heal, Argo CD quietly
  scales it back within minutes. Powerful — and
  occasionally surprising during incidents, when someone
  *needs* a hand-edit to survive.

## Sync waves (ordering)

Some resources must exist before others (a namespace
before the deployment in it, a CRD before the custom
resource). Argo CD mostly figures this out, but you can
be explicit with the `argocd.argoproj.io/sync-wave`
annotation — lower waves sync first:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Negative waves run before the default (wave 0). Use waves
sparingly — every explicit ordering is a dependency you're
now maintaining by hand.

## Sync options you'll actually use

- **`CreateNamespace=true`** — let Argo CD create the
  destination namespace instead of requiring it up front.
  Set as an annotation or a sync option; nearly everyone
  wants this on.
- **Selective sync** — in the UI, sync just one resource
  instead of the whole app. Handy when one deployment is
  misbehaving and you don't want to touch the rest.

## What "OutOfSync" really means

The app controller compares three things: git (desired),
the cluster (live), and its cache. OutOfSync means desired
≠ live. Common causes, in order: someone committed (sync
to catch up), someone hand-edited (sync to revert, or
update git to bless the change), or a controller
misunderstood a field (some fields like `status` or
auto-generated values need ignore rules — see
[Troubleshooting](troubleshooting.md)).

## The discipline

Automated sync without prune and self-heal is half a
GitOps loop — changes flow in, but deletions and drift
linger. Manual sync without anyone watching the UI is no
loop at all. Pick the combination your team will actually
maintain, and make the choice explicit per app. "We didn't
decide" always degrades to "nothing syncs and nobody
notices."
