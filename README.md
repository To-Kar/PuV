# PuV
Repo für Parallele und Verteilte Systeme Lab

## Voraussetzungen

Auf dem Zielrechner müssen installiert sein:

- **Docker Desktop** (Mac / Windows / Linux)
- **Kubernetes in Docker Desktop aktiviert**
- `kubectl` (kommt mit Docker Desktop)

> ❌ Kein Java, Maven, Node.js oder Postgres lokal nötig

---

## 1. Repository klonen

```bash
git clone <DEIN_REPO_URL>
cd <DEIN_REPO_NAME>
```

---

## 2. Kubernetes in Docker Desktop aktivieren

1. Docker Desktop öffnen
2. **Settings → Kubernetes**
3. ✅ **Enable Kubernetes** aktivieren
4. Warten bis **Kubernetes: Running** angezeigt wird

Danach prüfen:

```bash
kubectl config use-context docker-desktop
kubectl cluster-info
```

---

## 3. Docker Images lokal bauen

### Backend (Spring Boot)

```bash
docker build -t starterapp:v1 .
```

### Frontend (Vue + nginx)

```bash
cd frontend
docker build -t frontend:v1 .
cd ..
```

> Docker Desktop stellt diese Images automatisch Kubernetes zur Verfügung.

---

## 4. Kubernetes Deployments starten

```bash
kubectl apply -f k8s/postgres.yaml
kubectl apply -f k8s/backend.yaml
kubectl apply -f k8s/frontend.yaml
```

Status prüfen:

```bash
kubectl get pods -w
```

Warten, bis **alle Pods `Running`** sind.

---

## 5. App im Browser öffnen

### Frontend

```bash
kubectl port-forward service/frontend 5173:80
```

Browser:
```
http://localhost:5173
```

### Backend (optional, API direkt testen)

```bash
kubectl port-forward service/starterapp 8080:8080
```

API:
```
http://localhost:8080/todos
```

---

## App stoppen

### Variante A: nur pausieren (empfohlen)

```bash
kubectl scale deployment frontend --replicas=0
kubectl scale deployment starterapp --replicas=0
kubectl scale deployment postgresdb --replicas=0
```

Wieder starten:

```bash
kubectl scale deployment postgresdb --replicas=1
kubectl scale deployment starterapp --replicas=1
kubectl scale deployment frontend --replicas=1
```

---

### Variante B: alles löschen

```bash
kubectl delete -f k8s/frontend.yaml
kubectl delete -f k8s/backend.yaml
kubectl delete -f k8s/postgres.yaml
```
