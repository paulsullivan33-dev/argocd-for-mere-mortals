# Installing

You need a Kubernetes cluster. For learning, minikube or
kind on your own machine is plenty — nothing here touches
production.

## Install Argo CD

**Check:** the official install is one manifest, verified
October 2026:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

The `stable` URL tracks the latest stable release (v3.x
line as of this writing). For a pinned version, replace
`stable` with a tag like `v3.5.4` — check the [releases
page](https://github.com/argoproj/argo-cd/releases) for
what's current.

Wait for the pods:

```bash
kubectl get pods -n argocd
```

All should reach Running. On a small local cluster this
takes a few minutes — Argo CD is several services, not
one binary.

## Install the CLI

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

(Windows and macOS binaries are on the same releases
page. Package managers carry it too, usually a version
behind.)

## First login

The UI isn't exposed by default. Port-forward it:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open `https://localhost:8080`. The username is
`admin`; the initial password lives in a secret:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

**Change that password immediately** — via the UI (User
Info → Update Password) or `argocd account update-password`.
The initial secret is deleted after first login in newer
versions, but don't rely on that.

Log the CLI in the same way:

```bash
argocd login localhost:8080 --username admin --insecure
```

(`--insecure` skips TLS verification for your local
port-forward. Never use it against a real server.)

## What you should see

An empty UI with a "New App" button and no applications.
That's correct — Argo CD manages *applications you
define*; out of the box it manages nothing, which is
exactly the safe default you want. [Your first
app](your-first-app.md) fills that empty screen.
