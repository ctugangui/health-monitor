# Session Log

A running record of each working session on `health-monitor`: what got done, what broke, how it was fixed, and what comes next. Newest session at the bottom.

---

## Session 01: Environment setup

**Date:** 2026-10-14
**Goal:** Install the Go and Kubernetes toolchain, run a local cluster, and create the repo with a working hello-world.
**Result:** ✅ Done

### Definition of done

- [x] Go, Kind, kubectl, and Helm installed
- [x] Local cluster running, with `kubectl get nodes` showing `Ready`
- [x] Repo created on GitHub, cloned locally, first commit pushed
- [x] `go run .` prints `health-monitor v0.0`

### Steps

1. **Installed the toolchain** with Homebrew.
   ```bash
   brew install go kind kubectl helm
   ```
2. **Confirmed Docker was running.** Kind runs each Kubernetes node as a Docker container, so it needs a container runtime.
   ```bash
   docker ps
   ```
3. **Created the repo** on github.com with no README, .gitignore, or license, then cloned it.
   ```bash
   cd ~/development/golang
   git clone git@github.com:ctugangui/health-monitor.git
   cd health-monitor
   ```
4. **Initialized the Go module** and added `main.go`, `README.md`, and `.gitignore`.
   ```bash
   go mod init github.com/ctugangui/health-monitor
   go run .
   ```
5. **Made the first commit and pushed.**
   ```bash
   git add .
   git commit -m "init: go module and hello world"
   git branch -M main
   git push -u origin main
   ```
6. **Created the local cluster** and waited for the node to become ready.
   ```bash
   kind create cluster --name dev
   kubectl wait --for=condition=Ready node --all --timeout=90s
   kubectl get nodes
   ```
7. **Deleted the cluster** at the end of the session, since v0.1 doesn't need it.
   ```bash
   kind delete cluster --name dev
   ```

### Problems and fixes

| Symptom | Cause | Fix |
|---|---|---|
| `error: src refspec main does not match any` on push | The repo had no commits yet, so the `main` branch didn't exist | Commit first, then push |
| Local folder in a confusing state after `git init` plus `git clone` | Two ways of creating the repo got mixed together | Deleted the local folder, cloned the empty GitHub repo fresh, and added files there |
| `kubectl get nodes` → `connection refused` on `localhost:8080` | No cluster existed yet, so kubectl had no kubeconfig and fell back to its default address | Ran `kind create cluster`, which writes the kubeconfig entry |
| Node showed `NotReady` right after creation | The node waits for its network plugin (CNI) to start before reporting ready | Waited about 30 seconds; it changed to `Ready` |

### Concepts learned

- **Kubernetes** keeps containers running to a desired state. It restarts crashed ones and moves work off failed machines. The machines it manages are **nodes**.
- **Kind** (Kubernetes IN Docker) runs a real Kubernetes cluster locally, with each node as a Docker container. It's free and disposable, and a cluster takes about a minute to recreate.
- **kubectl** is the command-line client. It sends requests to the cluster's **API server** and doesn't run anything itself. It's the Kubernetes equivalent of the `aws` CLI.
- **The kubeconfig** (`~/.kube/config`) tells kubectl which cluster to talk to and how. A **context** names one cluster plus credentials. Kind created and selected `kind-dev`.
- **CNI** (Container Network Interface) gives pods their IP addresses. A node stays `NotReady` until its CNI is running, which makes "node NotReady" one of the most common Kubernetes debugging questions.
- **The control plane** is the cluster's brain: the API server, scheduler, and controllers. In Kind's single-node setup, one node acts as both control plane and worker.

### Decisions

- **Project location:** `~/development/golang/health-monitor`, next to the existing Python projects. Go modules don't depend on folder location.
- **Repo visibility:** public, as portfolio evidence.
- **Container runtime:** the existing Docker install. Colima wasn't needed.
- **Cluster lifecycle:** delete the cluster when it's not in use to free CPU and memory, and recreate it with `kind create cluster --name dev` when needed.

### Next session

**v0.1:** a Go program that checks a list of URLs concurrently and reports status and latency for each. It introduces goroutines, channels, and `net/http`, and runs locally without the cluster.

---

## Session template

Copy this block for each new session.

```markdown
## Session NN: Title

**Date:** YYYY-MM-DD
**Goal:** One sentence.
**Result:** ✅ Done / 🟡 Partial / ❌ Blocked

### Definition of done
- [ ] ...

### Steps
1. ...

### Problems and fixes
| Symptom | Cause | Fix |
|---|---|---|

### Concepts learned
- ...

### Decisions
- ...

### Next session
...
```
