# Helm Values

```bash

# Wert aus value.yaml nutzen
{{ .Values.service.name }} 

# Infos zur Version, chart, Kubernetes API
{{ .Release.Name }}

"{{ .Chart.Name }}-{{ .Chart.Version }}"

{{- if .Capabilities.APIVersions.Has "autoscaling/v2" }}
apiVersion: autoscaling/v2
{{- else }}
apiVersion: autoscaling/v1
{{- end }}

# Namen des aktuellen Templates zurückgeben 
metadata:
  annotations:
    helm.sh/template: {{ .Template.Name }}
```


# Control-Statements

```bash
# if
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
{{- end }}

# else
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
{{- else }}
// Service ist deaktiviert
{{- end }}

# with (Bereich von Werten)
{{- with .Values.service }}
name: {{ .name }}
type: {{ .type }}
{{- end }}

# range (Iterieren über Listen/Maps)
{{- range .Values.ports }}
- port: {{ . }}
{{- end }}
```

# Funktionen

```bash
# default-Wert
replicaCount: {{ .Values.replicaCount | default 1 }}

# Escapen (Modifizieren von Zeichen, damit sie als Yaml interpretiert werden können)
env: {{ .Values.env | quote }}

# required (wirft Fehler, falls kein Wert vorhanden)
image: {{ required "Das Image muss gesetzt sein!" .Values.image }}

# whitespaces links der Anwendung entfernen 
# (notwendig, da bei Nichterfüllung Leerzeile entstehen würde)
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
{{- end }}

# tpl Ausführen eines Strings als Template
annotations:
  description: {{ tpl .Values.description . }}

# include (Einbinden anderer Templates)
{{ include "mychart.helpers.some-function" . }}

# lookup (externe Ressourcen wie CM oder SVC im Template benutzen)
{{- $svc := lookup "v1" "Service" "default" "my-service" -}}

```

# Lokales Rendern von Templates 

```bash
helm template <release-name> <chart-path> [flags]

# spezifische Values
helm template <release-name> <chart-path> -f <values-file1> -f <values-file2> 

# setzen von Werten
helm template <release-name> <chart-path> --set key1=value1,key2=value2

# nur bestimmte Templates rendern
helm template <release-name> <chart-path> --show-only templates/<template-file>

```

# Pull und Push von HELM Charts

```bash
# in artifact hub nach charts suchen:
helm search hub wordpress --output yaml # Funktioniert ohne hinzufügen eines lokalen repos

# Helm-Chart von einem Repo pullen und installieren
helm repo add bitnami https://charts.bitnami.com/bitnami        
helm repo update
helm repo list # alle hinzugefügten Repos anzeigen
helm search repo bitnami # nur im hinzugefügten Repo bitnami suchen
helm install my-release bitnami/nginx
helm list # installierte Releases anzeigen
helm pull bitnami/nginx

# Helm-Charts in eigenen Harbor pushen
helm registry login https://craas-1a.itz.cloud.intranet.bund.de/
helm push nginx-13.2.8.tgz oci://craas-1a.itz.cloud.intranet.bund.de/import-itzbund-fms-devops-tools




```
