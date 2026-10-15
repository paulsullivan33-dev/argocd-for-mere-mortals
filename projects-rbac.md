# Projects and RBAC

Out of the box, everything lives in the `default`
AppProject and the admin can do anything. That stops being
fine the moment a second team — or a second environment —
shows up.

## AppProjects (bulkheads)

An **AppProject** is a named boundary: which repos, which
clusters, and which destination namespaces a set of
Applications may use. Think of it as the bulkhead that
keeps team A's app from deploying into team B's namespace.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  description: Team A's workloads
  sourceRepos:
    - https://github.com/org/team-a-apps.git
  destinations:
    - namespace: team-a-*
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

Key fields:

- **`sourceRepos`** — which git repos this project's apps
  may pull from. `*` allows all; don't, once you have
  more than one team.
- **`destinations`** — where apps may deploy. Namespace
  globs (`team-a-*`) keep teams in their lanes.
- **`clusterResourceWhitelist`** — cluster-scoped
  resources (ClusterRoles, CRDs) are denied by default;
  whitelist what a project genuinely needs.

Applications opt in with `spec.project: team-a`. An app
whose repo or destination falls outside its project is
rejected — which is the point.

## RBAC (who may do what)

Argo CD has its own RBAC, configured in `argocd-rbac-cm`,
separate from Kubernetes RBAC. The default roles:

- **`role:admin`** — everything.
- **`role:readonly`** — look, don't touch.

Real setups define narrower roles. The ConfigMap syntax:

```ini
# argocd-rbac-cm
data:
  policy.csv: |
    p, role:deployer, applications, sync, team-a/*, allow
    p, role:deployer, applications, get, team-a/*, allow
    g, team-a-devs, role:deployer
```

Translation: members of `team-a-devs` (mapped from your
SSO groups via Dex, or local users) may sync and view apps
in project `team-a`, and nothing else.

## The sane progression

1. **Solo learning:** default project, admin user. Fine.
2. **One team, staging + prod:** two projects
   (`staging`, `prod`), deployers sync staging freely,
   prod requires the narrower role — or manual sync with
   a second pair of eyes.
3. **Many teams:** one project per team, SSO groups
   mapped to roles, cluster resources whitelisted
   sparingly.

Don't build step 3 on day one. RBAC nobody understands is
worse than no RBAC — it just produces "permission denied"
tickets and a shared admin password written on a wiki.
