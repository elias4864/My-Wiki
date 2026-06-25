# My-Wiki - Wiki.js mit Kubernetes und Helm

Dieses Projekt deployt Wiki.js als Dokumentenmanagement- und Wissensplattform in einem Kubernetes-Cluster. Wiki.js laeuft als Webapplikation und nutzt PostgreSQL als Datenbank.

## Komponenten

- Wiki.js Webapplikation
- PostgreSQL Datenbank
- Kubernetes Service fuer Wiki.js
- Kubernetes Service fuer PostgreSQL
- PersistentVolumeClaim fuer PostgreSQL-Daten
- Optionaler Ingress fuer `wiki.minikube.local`
- GitHub Actions Pipeline fuer Helm-Pruefung

## Voraussetzungen

Auf dem Rechner muessen diese Tools installiert sein:

- Docker
- Minikube
- kubectl
- Helm
- Git

Versionen pruefen:

```powershell
docker --version
minikube version
kubectl version --client
helm version
git --version
```

## Projektordner oeffnen

```powershell
cd "C:\Users\fabia\OneDrive - Kalaidos Bildungsgruppe AG\wiss\210\My-Wiki"
```

## Erstinstallation

Minikube starten:

```powershell
minikube start --cpus 2 --memory 4096
```

Namespace erstellen:

```powershell
kubectl create namespace wiki
```

Helm-Chart installieren:

```powershell
helm install wiki . -n wiki
```

Status pruefen:

```powershell
kubectl get pods -n wiki
kubectl get svc -n wiki
kubectl get pvc -n wiki
```

Erwartet:

```text
wikijs-...              1/1 Running
wikijs-postgresql-...   1/1 Running
```

## Normaler Start nach bereits erfolgter Installation

Wenn das Projekt schon einmal installiert wurde, muss nicht jedes Mal `helm install` ausgefuehrt werden.

Minikube starten:

```powershell
minikube start
```

Pods pruefen:

```powershell
kubectl get pods -n wiki
```

Zugriff starten:

```powershell
kubectl port-forward svc/wikijs-service 8081:3000 -n wiki
```

Browser oeffnen:

```text
http://localhost:8081
```

Das PowerShell-Fenster mit `kubectl port-forward` muss offen bleiben.

## Nach Aenderungen am Helm-Chart

Wenn Dateien im Chart geaendert wurden, wird das bestehende Deployment aktualisiert:

```powershell
helm upgrade wiki . -n wiki
```

Danach pruefen:

```powershell
kubectl get pods -n wiki
```

## Zugriff per Port-Forwarding

Der Service ist als `ClusterIP` konfiguriert. Deshalb ist Wiki.js nicht automatisch von Windows aus erreichbar. Fuer Tests wird Port-Forwarding verwendet:

```powershell
kubectl port-forward svc/wikijs-service 8081:3000 -n wiki
```

Danach:

```text
http://localhost:8081
```

## Optionaler Zugriff per Ingress

Ingress ist vorbereitet, aber optional. Fuer Minikube muss zuerst das Ingress Addon aktiviert werden:

```powershell
minikube addons enable ingress
```

Minikube-IP anzeigen:

```powershell
minikube ip
```

Danach muss in der Windows-Hosts-Datei ein Eintrag erstellt werden:

```text
<MINIKUBE-IP> wiki.minikube.local
```

Dann ist Wiki.js ueber diese Adresse erreichbar:

```text
http://wiki.minikube.local
```

Fuer die Abgabe und einfache Demo ist Port-Forwarding ausreichend.

## Helm-Chart pruefen

Syntax und Chart-Struktur pruefen:

```powershell
helm lint .
```

Kubernetes-Manifeste rendern:

```powershell
helm template wiki .
```

## Logs pruefen

Wiki.js Logs:

```powershell
kubectl logs deploy/wikijs -n wiki
```

PostgreSQL Logs:

```powershell
kubectl logs deploy/wikijs-postgresql -n wiki
```

## Persistenz testen

Eine Wiki-Seite in Wiki.js erstellen. Danach den Wiki.js Pod neu starten:

```powershell
kubectl delete pod -l app=wikijs -n wiki
kubectl get pods -n wiki
```

Danach wieder im Browser oeffnen und pruefen, ob die Seite noch vorhanden ist:

```text
http://localhost:8081
```

Die Seite sollte erhalten bleiben, weil die Daten in PostgreSQL gespeichert werden.

## Deinstallation

Helm Release entfernen:

```powershell
helm uninstall wiki -n wiki
```

Namespace komplett entfernen:

```powershell
kubectl delete namespace wiki
```

## Troubleshooting

### Pod ist `Pending`

Status und Events pruefen:

```powershell
kubectl get pods -n wiki -o wide
kubectl get events -n wiki --sort-by=.lastTimestamp
```

### Pod ist `0/1 Running`

Der Container laeuft, ist aber noch nicht ready. Logs pruefen:

```powershell
kubectl logs deploy/wikijs -n wiki
kubectl describe pod -l app=wikijs -n wiki
```

### Port-Forwarding funktioniert nicht

Pruefen, ob Wiki.js ready ist:

```powershell
kubectl get pods -n wiki
```

Wenn `wikijs` noch nicht `1/1 Running` ist, kurz warten und erneut pruefen.

### Namespace existiert schon

Wenn dieser Befehl einen Fehler zeigt:

```powershell
kubectl create namespace wiki
```

dann existiert der Namespace bereits. In diesem Fall kann direkt mit Helm weitergearbeitet werden.

## Pipeline

Die GitHub Actions Pipeline liegt unter:

```text
.github/workflows/helm-check.yml
```

Sie laeuft bei Push und Pull Request auf `main` und `dev`. Sie prueft:

```bash
helm lint .
helm template wiki .
```
