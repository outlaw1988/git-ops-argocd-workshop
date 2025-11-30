# GitOps Demo – ArgoCD (dev/stage/prod + App-of-Apps)

To repo demonstruje podejście GitOps z użyciem **ArgoCD**:

- struktura `apps/` (aplikacje) + `clusters/` (środowiska),
- środowiska: **dev**, **stage**, **prod**,
- wzorzec **App-of-Apps** – jedna aplikacja ArgoCD na środowisko,
- Kustomize overlays per środowisko.

## 1. Wymagania

- Kubernetes (Minikube lub Docker Desktop z włączonym Kubernetes),
- `kubectl`,
- ArgoCD zainstalowany w namespace `argocd`,
- (opcjonalnie) `argocd` CLI.

### 1.1. Sprawdzenie klastra

```bash
kubectl get nodes
```

Powinien być przynajmniej 1 node w stanie `Ready`.

## 2. Klonowanie repo

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/gitops-demo.git
cd gitops-demo
```

> **Uwaga:** Zmień `YOUR_GITHUB_USERNAME` w plikach:
> - `clusters/dev/apps-root.yaml`
> - `clusters/stage/apps-root.yaml`
> - `clusters/prod/apps-root.yaml`
> - `clusters/*/apps/guestbook-app.yaml`
> - `clusters/*/apps/podinfo-app.yaml`

na swój login / URL repo.

## 3. Struktura repo

```text
apps/
  guestbook/
    base/
    overlays/
      dev/
      stage/
      prod/
  podinfo/
    base/
    overlays/
      dev/
      stage/
      prod/
clusters/
  dev/
    apps-root.yaml
    apps/
      guestbook-app.yaml
      podinfo-app.yaml
  stage/
    apps-root.yaml
    apps/
      guestbook-app.yaml
      podinfo-app.yaml
  prod/
    apps-root.yaml
    apps/
      guestbook-app.yaml
      podinfo-app.yaml
```

- `apps/*/base` – wspólny manifest dla wszystkich środowisk,
- `apps/*/overlays/*` – różnice środowisk (namespace, etykiety, itp.),
- `clusters/<env>/apps-root.yaml` – root Application (App-of-Apps),
- `clusters/<env>/apps/*.yaml` – Applications dla poszczególnych aplikacji.

## 4. Uruchomienie środowiska **dev** (App-of-Apps)

1. Zastosuj root Application:

```bash
kubectl apply -f clusters/dev/apps-root.yaml
```

2. W ArgoCD UI powinna pojawić się aplikacja `apps-dev`.
3. Po synchronizacji `apps-dev` utworzy aplikacje:
   - `guestbook-dev`
   - `podinfo-dev`

Możesz też na DEV zrobić ręcznie:

```bash
kubectl apply -f clusters/dev/apps/guestbook-app.yaml
kubectl apply -f clusters/dev/apps/podinfo-app.yaml
```

## 5. Uruchomienie **stage** i **prod**

Analogicznie:

```bash
kubectl apply -f clusters/stage/apps-root.yaml
kubectl apply -f clusters/prod/apps-root.yaml
```

Lub pojedyncze aplikacje:

```bash
kubectl apply -f clusters/stage/apps/guestbook-app.yaml
kubectl apply -f clusters/prod/apps/guestbook-app.yaml
```

## 6. Ćwiczenia dla uczestników

### Ćwiczenie 1 – Zmiana liczby replik na DEV

1. Otwórz plik:
   - `apps/guestbook/base/deployment.yaml`
2. Zmień:

```yaml
spec:
  replicas: 2
```

na np.:

```yaml
spec:
  replicas: 4
```

3. Zrób `git commit` + `git push` do swojego repo.
4. W ArgoCD zobaczysz:
   - aplikacja `guestbook-dev` jest `OutOfSync`,
   - po synchronizacji liczba podów wzrośnie.

### Ćwiczenie 2 – Różne komunikaty dla DEV/STAGE/PROD

1. W plikach overlays (np. patches) ustaw różne wartości zmiennej env:
   - DEV: "Hello from DEV",
   - STAGE: "Hello from STAGE",
   - PROD: "Hello from PROD".
2. Wdróż wszystkie środowiska i pokaż różnice w UI aplikacji.

### Ćwiczenie 3 – Rollback

1. Zmień manifest tak, aby aplikacja się nie podniosła (np. zły port, zła nazwa obrazu).
2. Zobacz w ArgoCD status `Degraded`.
3. Użyj funkcji **Rollback** w UI, aby wrócić do poprzedniej, działającej wersji.

## 7. Sprzątanie

```bash
kubectl delete -f clusters/dev/apps-root.yaml
kubectl delete -f clusters/stage/apps-root.yaml
kubectl delete -f clusters/prod/apps-root.yaml

# lub usuwaj pojedyncze aplikacje
kubectl delete -f clusters/dev/apps/guestbook-app.yaml
kubectl delete -f clusters/dev/apps/podinfo-app.yaml

# oraz (opcjonalnie) namespace’y
kubectl delete ns dev stage prod
```