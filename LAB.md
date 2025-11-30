# ArgoCD Installation Lab

## 1. Utwórz namespace

```bash
kubectl create namespace argocd
```

## 2. Zastosuj oficjalną instalację (stable manifests)

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 3. Sprawdź, czy komponenty działają

```bash
kubectl get pods -n argocd
```

## 4. Wystaw ArgoCD UI (port-forward)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

UI będzie dostępne pod:
- https://localhost:8080

## 5. Pobierz hasło admina

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

**Login:** `admin`
