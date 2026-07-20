# Helm Referenz – Kubernetes & OpenShift

> Kompakte Referenz zum schnellen Nachschlagen von Helm-Befehlen, Templates und typischen Workflows (OpenShift/EX280).

## Inhaltsverzeichnis

- [1. Grundlagen](#1-grundlagen)
- [2. Repositorys](#2-repositorys)
- [3. Charts suchen](#3-charts-suchen)
- [4. Installieren](#4-installieren)
- [5. Upgrade & Rollback](#5-upgrade--rollback)
- [6. Releases verwalten](#6-releases-verwalten)
- [7. values.yaml](#7-valuesyaml)
- [8. Template-Rendering](#8-template-rendering)
- [9. Go-Template Referenz](#9-go-template-referenz)
- [10. Control Statements](#10-control-statements)
- [11. Wichtige Funktionen](#11-wichtige-funktionen)
- [12. _helpers.tpl](#12-_helperstpl)
- [13. OCI Registry](#13-oci-registry)
- [14. Debugging](#14-debugging)
- [15. OpenShift Tipps](#15-openshift-tipps)
- [16. Troubleshooting](#16-troubleshooting)
- [17. Cheatsheet](#17-cheatsheet)

---

# 1. Grundlagen

| Begriff | Bedeutung |
|---|---|
| Chart | Helm-Paket |
| Release | Installierte Instanz eines Charts |
| Repository | Quelle für Charts |
| values.yaml | Konfiguration |
| Template | Kubernetes YAML mit Go-Templates |

```text
Chart
  │
helm install
  │
Release
  │
Deployment / Service / Secret / Route ...
```

💡 **Merke:** Ein Chart kann beliebig oft installiert werden. Jede Installation ist ein eigenes **Release**.

---

# 2. Repositorys

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list
helm repo remove bitnami
```

---

# 3. Charts suchen

```bash
helm search repo nginx                    # nginx kann auch weggelassen werden, da wird alles ausgegeben
helm search repo nginx --versions
helm search hub wordpress
```

---

# 4. Installieren

```bash
helm install myapp bitnami/nginx
```

Bestimmte Version

```bash
helm install myapp do280/etherpad --version 0.0.7
```

Namespace

```bash
helm install myapp bitnami/nginx -n demo --create-namespace
```

Values

```bash
helm install myapp do280/etherpad -f values.yaml -f values2.yaml
```

Einzelne Werte

```bash
helm install myapp do280/etherpad \
  --set replicaCount=3 \
  --set image.tag=2.0
```

---

# 5. Upgrade & Rollback

```bash
helm upgrade myapp do280/etherpad -f values.yaml
```

Upgrade oder Install:

```bash
helm upgrade myapp do280/etherpad \
  --install \
  -f values.yaml
```

Dry Run

```bash
helm upgrade myapp do280/etherpad \
  --dry-run --debug
```

Rollback

```bash
helm history myapp
helm rollback myapp 2
```

---

# 6. Releases verwalten

```bash
helm list
helm list -A
helm status myapp
helm history myapp
helm uninstall myapp
```

Manifest anzeigen

```bash
helm get manifest myapp
```

Werte anzeigen

```bash
helm get values myapp
helm get values myapp --all
```

---

# 7. values.yaml

Priorität (unten überschreibt oben)

```text
Chart values.yaml
      ↓
Parent values.yaml
      ↓
-f values1.yaml
      ↓
-f values2.yaml
      ↓
--set
      ↓
--set-string
```

Mehrere Dateien

```bash
helm upgrade myapp chart \
-f common.yaml \
-f prod.yaml
```

---

# 8. Template Rendering

Nur rendern

```bash
helm template myapp do280/etherpad
```

Mit Values

```bash
helm template myapp do280/etherpad \
-f values.yaml
```

Nur ein Template

```bash
helm template myapp chart \
--show-only templates/service.yaml
```

Mit Cluster vergleichen

```bash
helm template myapp chart \
| oc diff -f -
```

⚠️ **Nicht** `helm upgrade --dry-run | oc diff`, da `upgrade` zusätzliche Statusinformationen ausgibt.

---

# 9. Go-Template Referenz

## .Values

```gotemplate
{{ .Values.service.name }}
```

## .Release

```gotemplate
{{ .Release.Name }}
{{ .Release.Namespace }}
```

## .Chart

```gotemplate
{{ .Chart.Name }}
{{ .Chart.Version }}
```

## .Capabilities

```gotemplate
{{- if .Capabilities.APIVersions.Has "autoscaling/v2" }}
apiVersion: autoscaling/v2
{{- else }}
apiVersion: autoscaling/v1
{{- end }}
```

## .Template

```gotemplate
{{ .Template.Name }}
```

---

# 10. Control Statements

## if

```gotemplate
{{- if .Values.service.enabled }}
kind: Service
{{- end }}
```

## if / else

```gotemplate
{{- if .Values.service.enabled }}
enabled
{{- else }}
disabled
{{- end }}
```

## with

```gotemplate
{{- with .Values.service }}
name: {{ .name }}
type: {{ .type }}
{{- end }}
```

## range

```gotemplate
{{- range .Values.ports }}
- port: {{ . }}
{{- end }}
```

---

# 11. Wichtige Funktionen

## default

```gotemplate
{{ .Values.replicaCount | default 1 }}
```

## required

```gotemplate
{{ required "Image fehlt!" .Values.image }}
```

## quote

```gotemplate
{{ .Values.env | quote }}
```

## include

```gotemplate
{{ include "mychart.fullname" . }}
```

## tpl

```gotemplate
{{ tpl .Values.description . }}
```

## lookup

```gotemplate
{{- $svc := lookup "v1" "Service" "default" "mysql" -}}
```

> Funktioniert nur mit Clusterzugriff.

## toYaml + nindent

```gotemplate
resources:
{{- toYaml .Values.resources | nindent 2 }}
```

## indent

```gotemplate
{{ include "labels" . | indent 4 }}
```

## b64enc

```gotemplate
{{ .Values.password | b64enc }}
```

---

# 12. _helpers.tpl

Definition

```gotemplate
{{- define "mychart.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

Verwendung

```gotemplate
metadata:
  name: {{ include "mychart.fullname" . }}
```

---

# 13. OCI Registry

Login

```bash
helm registry login registry.example.com
```

Pull

```bash
helm pull oci://registry.example.com/charts/nginx
```

Push

```bash
helm push nginx-1.0.0.tgz \
oci://registry.example.com/charts
```

Harbor/OpenShift Registry funktionieren ebenfalls über OCI.

---

# 14. Debugging

Chart prüfen

```bash
helm lint .
```

Templates rendern

```bash
helm template myapp .
```

Chart Informationen

```bash
helm show chart chart
helm show values chart
helm show readme chart
```

Dependencies

```bash
helm dependency list .
helm dependency update .
```

---

# 15. OpenShift Tipps

Rendern und vergleichen

```bash
helm template myapp chart \
| oc diff -f -
```

Installieren

```bash
helm upgrade myapp chart \
--install \
-n demo
```

GitOps (Argo CD) verwendet typischerweise:

- Chart
- values.yaml
- valueFiles
- OCI Charts

---

# 16. Troubleshooting

| Problem | Ursache | Lösung |
|---|---|---|
| Chart not found | Repo fehlt | `helm repo update` |
| Unknown version | Falsche Version | `helm search repo --versions` |
| Values greifen nicht | Reihenfolge | `helm get values --all` |
| YAML Fehler | Template | `helm template --debug` |
| Release existiert | install statt upgrade | `helm upgrade --install` |

---

# 17. Cheatsheet

```bash
# Repositories
helm repo add
helm repo update
helm repo list

# Suche
helm search repo nginx --versions

# Install
helm install RELEASE CHART

# Upgrade
helm upgrade RELEASE CHART
helm upgrade RELEASE CHART --install

# Rollback
helm history RELEASE
helm rollback RELEASE REV

# Releases
helm list -A
helm status RELEASE

# Rendering
helm template RELEASE CHART
helm template RELEASE CHART -f values.yaml
helm template RELEASE CHART | oc diff -f -

# Values
helm show values CHART
helm get values RELEASE --all

# Debug
helm lint .
helm show chart CHART
helm get manifest RELEASE

# OCI
helm registry login REGISTRY
helm pull oci://REGISTRY/CHART
helm push chart.tgz oci://REGISTRY
```

## EX280 Merksätze

- `helm template` für Vergleiche mit `oc diff`.
- Chart = Paket, Release = Installation.
- `values.yaml` + `-f` + `--set` sicher beherrschen.
- `include`, `default`, `required`, `toYaml`, `nindent` gehören zu den wichtigsten Template-Funktionen.
- `_helpers.tpl` enthält wiederverwendbare Template-Funktionen.
