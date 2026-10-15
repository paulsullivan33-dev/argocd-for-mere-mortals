# Glossary

**Application** — Argo CD's core custom resource: a repo,
a path, and a destination. The unit of GitOps.

**app-of-apps** — a parent Application whose manifests are
other Applications. Bootstraps a whole cluster from one
apply.

**ApplicationSet** — generates Applications from
templates (per cluster, per environment). App-of-apps
grown up.

**AppProject** — a boundary: which repos, clusters, and
namespaces a set of Applications may use.

**automated sync** — Argo CD applies detected changes
without human approval. Pair with prune and self-heal
deliberately.

**desired state** — what git says should be running. The
"truth" half of the comparison.

**drift** — the cluster disagreeing with git. Shown as
OutOfSync; the thing GitOps exists to eliminate.

**GitOps** — the practice: git is the source of truth,
software agents reconcile reality to it. Argo CD is a
GitOps *operator*, not the idea itself.

**ignoreDifferences** — per-app rules telling Argo CD
which live-vs-git field diffs don't count (HPA-managed
replicas, injected defaults).

**live state** — what's actually running in the cluster.
The "reality" half of the comparison.

**manual sync** — changes apply only when a human presses
Sync. The safe default.

**prune** — deleting cluster resources that were removed
from git. The removal half of "git is the truth."

**RBAC** — Argo CD's own access control
(`argocd-rbac-cm`), separate from Kubernetes RBAC.
Defines who may sync, view, or administer what.

**repo server** — the Argo CD component that clones git
repos and renders manifests (YAML, Helm, Kustomize).

**self-heal** — automatically reverting hand-edits back
to git state. Powerful; surprising during incidents.

**sync** — Argo CD applying git's desired state to the
cluster.

**sync wave** — an annotation ordering which resources
sync first (lower waves first; negative before zero).

**targetRevision** — the branch, tag, or commit an
Application tracks. `HEAD` follows the branch tip.
