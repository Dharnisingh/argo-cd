# Deploy to Minikube using Argo CD (Helm)

1. Push this repository to a public GitHub repo (or a repo Argo CD can access). Replace <YOUR_GIT_REPO> in `application/argocd/speedtest-app.yaml` with the repo URL.

2. Start minikube:

```bash
minikube start --driver=docker
```

3. (Optional) If you want to use a local image instead of pushing to a registry, build and load it into minikube:

```bash
# from repo root
docker build -t dharnisingh/speedtest:latest -f application/dockerfile .
minikube image load dharnisingh/speedtest:latest
```

4. Install Argo CD in the cluster:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

5. Expose the Argo CD API server (port-forward):

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

6. Get the initial admin password and login (or install `argocd` CLI):

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
# then
argocd login localhost:8080 --username admin --password <password> --insecure
```

7. Add your Git repo to Argo CD (if needed) and create the application, or apply the Application manifest after updating `repoURL`:

```bash
# using CLI (example)
argocd repo add https://github.com/<user>/<repo>.git
argocd app create speedtest-helm --repo https://github.com/<user>/<repo>.git --path application/helm/speedtest --dest-namespace default --dest-server https://kubernetes.default.svc --helm

# or, if Argo CD can access the repo, apply the manifest (ensure repoURL is correct):
kubectl apply -f application/argocd/speedtest-app.yaml -n argocd
```

8. Sync the application (via CLI, UI, or `kubectl`):

```bash
argocd app sync speedtest-helm
```

Notes:
- Ensure the repository is reachable by Argo CD (public or configured credentials).
- Replace `application/helm/speedtest/values.yaml` values if you need different image/tag/replicas.
