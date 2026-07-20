# Helm – Befehlsübersicht und Praxisbeispiele

> **Zweck:** Helm ist der Paketmanager für Kubernetes. Charts bündeln Kubernetes-Manifeste und ermöglichen reproduzierbare Installationen, Upgrades und Rollbacks.

## Inhaltsverzeichnis

1. Helm-Grundlagen
2. Repositories
3. Charts suchen
4. Release installieren
5. Releases verwalten
6. Upgrades
7. Rollbacks
8. Werte (values.yaml)
9. Templates rendern
10. Chart-Informationen
11. Packaging
12. OpenShift-Praxis
13. Häufige Fehler
14. Spickzettel

---

# 1. Helm-Grundlagen

## Wichtige Begriffe

| Begriff | Bedeutung |
|---------|-----------|
| Chart | Helm-Paket |
| Release | Installierte Instanz eines Charts |
| Repository | Sammlung von Charts |
| values.yaml | Konfigurationswerte |
| Template | Kubernetes-Manifeste mit Go-Templates |

Beispiel:

```text
Chart
   │
helm install
   │
Release
   │
Deployment
Service
Ingress ...
```

---

# 2. Repositories

Repository hinzufügen

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Repository aktualisieren

```bash
helm repo update
```

Repositories anzeigen

```bash
helm repo list
```

Repository entfernen

```bash
helm repo remove bitnami
```

---

# 3. Charts suchen

Remote suchen

```bash
helm search repo nginx
```

Alle Versionen

```bash
helm search repo nginx --versions
```

Lokale Charts

```bash
helm search hub wordpress
```

---

# 4. Release installieren

Standard

```bash
helm install my-nginx bitnami/nginx
```

Bestimmte Version

```bash
helm install myapp do280/etherpad \
  --version 0.0.7
```

Eigenes Namespace

```bash
helm install myapp bitnami/nginx \
  -n demo \
  --create-namespace
```

Mit values-Datei

```bash
helm install myapp do280/etherpad \
  -f values.yaml
```

Einzelne Werte überschreiben

```bash
helm install myapp do280/etherpad \
  --set replicaCount=3
```

Mehrere Werte

```bash
helm install myapp do280/etherpad \
  --set image.tag=2.0 \
  --set service.type=NodePort
```

---

# 5. Releases verwalten

Alle Releases

```bash
helm list
```

Alle Namespaces

```bash
helm list -A
```

Status

```bash
helm status myapp
```

Historie

```bash
helm history myapp
```

Deinstallieren

```bash
helm uninstall myapp
```

---

# 6. Upgrades

Normales Upgrade

```bash
helm upgrade myapp do280/etherpad \
  -f values.yaml
```

Bestimmte Version

```bash
helm upgrade myapp do280/etherpad \
  --version 0.0.7 \
  -f values.yaml
```

Installieren falls Release fehlt

```bash
helm upgrade myapp do280/etherpad \
  --install
```

Dry Run

```bash
helm upgrade myapp do280/etherpad \
  --dry-run \
  -f values.yaml
```

Debug

```bash
helm upgrade myapp do280/etherpad \
  --dry-run \
  --debug
```

---

# 7. Rollback

Historie

```bash
helm history myapp
```

Rollback

```bash
helm rollback myapp 2
```

---

# 8. values.yaml

Default-Werte anzeigen

```bash
helm show values do280/etherpad
```

Eigene Werte verwenden

```bash
helm install myapp do280/etherpad \
  -f values.yaml
```

Mehrere Dateien

```bash
helm upgrade myapp do280/etherpad \
  -f common.yaml \
  -f prod.yaml
```

Spätere Dateien überschreiben frühere.

---

# 9. Templates rendern

Nur rendern

```bash
helm template myapp do280/etherpad
```

Mit values

```bash
helm template myapp do280/etherpad \
  -f values.yaml
```

Rendern und mit Cluster vergleichen

```bash
helm template myapp do280/etherpad \
  -f values.yaml \
  | oc diff -f -
```

> `helm template` eignet sich besser für Pipes als `helm upgrade --dry-run`, da ausschließlich Kubernetes-Manifeste ausgegeben werden.

Renderte Manifeste einer Installation anzeigen

```bash
helm get manifest myapp
```

---

# 10. Chart-Informationen

Chart anzeigen

```bash
helm show chart do280/etherpad
```

README

```bash
helm show readme do280/etherpad
```

Alle Informationen

```bash
helm show all do280/etherpad
```

Abhängigkeiten

```bash
helm dependency list .
```

Aktualisieren

```bash
helm dependency update .
```

---

# 11. Packaging

Chart validieren

```bash
helm lint .
```

Chart paketieren

```bash
helm package .
```

Chart entpacken

```bash
helm pull do280/etherpad --untar
```

Chart als tgz laden

```bash
helm pull do280/etherpad
```

---

# 12. OpenShift-Praxis

Releases

```bash
helm list -A
```

Gerenderte YAML prüfen

```bash
helm template myapp do280/etherpad \
  -f values.yaml
```

Mit Cluster vergleichen

```bash
helm template myapp do280/etherpad \
  -f values.yaml \
  | oc diff -f -
```

Namespace explizit

```bash
helm upgrade myapp do280/etherpad \
  -n demo \
  -f values.yaml
```

Vorhandene Ressourcen übernehmen

```bash
helm upgrade myapp do280/etherpad \
  --install
```

---

# 13. Häufige Fehler

## repository not found

```text
Error: chart not found
```

Repository aktualisieren

```bash
helm repo update
```

---

## unknown chart version

Version prüfen

```bash
helm search repo do280/etherpad --versions
```

---

## values werden nicht übernommen

Kontrollieren

```bash
helm get values myapp
```

Alle Werte

```bash
helm get values myapp --all
```

---

## Dry Run liefert kein gültiges YAML

Falsch

```bash
helm upgrade myapp do280/etherpad \
  --dry-run \
| oc diff -f -
```

Richtig

```bash
helm template myapp do280/etherpad \
| oc diff -f -
```

---

## Lint verwenden

```bash
helm lint .
```

Findet viele Template-Fehler frühzeitig.

---

# 14. Spickzettel

```bash
# Repository
helm repo list
helm repo update

# Suchen
helm search repo nginx --versions

# Installieren
helm install RELEASE CHART

# Upgrade
helm upgrade RELEASE CHART

# Upgrade oder Install
helm upgrade RELEASE CHART --install

# Rollback
helm rollback RELEASE REVISION

# Releases
helm list -A

# Status
helm status RELEASE

# Historie
helm history RELEASE

# Rendern
helm template RELEASE CHART

# Default Values
helm show values CHART

# Manifest einer Installation
helm get manifest RELEASE

# Benutzerwerte
helm get values RELEASE --all

# Lint
helm lint .

# Paket bauen
helm package .

# Chart herunterladen
helm pull CHART --untar

# Diff mit OpenShift
helm template RELEASE CHART | oc diff -f -
```

## Tipps für EX280

- `helm template` statt `helm upgrade --dry-run` für `oc diff`
- Unterschied zwischen **Chart** und **Release** kennen.
- `values.yaml` und `--set` sicher beherrschen.
- `helm show values`, `helm get values` und `helm history` gehören zu den wichtigsten Diagnosebefehlen.
- Vor einem Upgrade mit `helm template` prüfen, welche Ressourcen erzeugt werden.
