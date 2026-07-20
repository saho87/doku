# Skopeo – Befehlsübersicht und Praxisbeispiele

> **Zweck:** Container-Images und Registries untersuchen, kopieren, synchronisieren und verwalten – ohne einen Container-Daemon und ohne das Image zunächst lokal zu pullen.

## Inhaltsverzeichnis

- [1. Was ist Skopeo?](#1-was-ist-skopeo)
- [2. Grundlegende Syntax](#2-grundlegende-syntax)
- [3. Installation und Hilfe](#3-installation-und-hilfe)
- [4. Images untersuchen](#4-images-untersuchen)
- [5. Tags eines Repositorys auflisten](#5-tags-eines-repositorys-auflisten)
- [6. An einer Registry anmelden](#6-an-einer-registry-anmelden)
- [7. Images kopieren](#7-images-kopieren)
- [8. Images lokal sichern und übertragen](#8-images-lokal-sichern-und-übertragen)
- [9. Repositorys synchronisieren](#9-repositorys-synchronisieren)
- [10. Images löschen](#10-images-löschen)
- [11. OpenShift-Praxis](#11-openshift-praxis)
- [12. TLS, Zertifikate und Authentifizierung](#12-tls-zertifikate-und-authentifizierung)
- [13. Nützliche Ausgaben mit jq](#13-nützliche-ausgaben-mit-jq)
- [14. Häufige Fehler](#14-häufige-fehler)
- [15. Skopeo, Podman und Buildah](#15-skopeo-podman-und-buildah)
- [16. Spickzettel](#16-spickzettel)

---

## 1. Was ist Skopeo?

Skopeo arbeitet direkt mit Container-Images und Container-Registries. Typische Anwendungsfälle sind:

- Metadaten eines Remote-Images anzeigen
- verfügbare Tags eines Repositorys auflisten
- Images zwischen Registries kopieren
- Images in lokale Verzeichnisse oder OCI-Archive übertragen
- Multi-Architektur-Images kopieren
- Registry-Inhalte spiegeln
- Images aus einer Registry löschen

Im Gegensatz zu `podman pull` muss `skopeo inspect` das Image nicht zunächst vollständig in den lokalen Container-Storage herunterladen.

---

## 2. Grundlegende Syntax

```bash
skopeo <COMMAND> [OPTIONEN] <IMAGE-REFERENZ>
```

Skopeo verwendet sogenannte **Transports**:

```text
<transport>:<image-reference>
```

Häufige Transports:

| Transport | Verwendung | Beispiel |
|---|---|---|
| `docker://` | Registry nach Docker Registry HTTP API V2 | `docker://quay.io/example/app:1.0` |
| `dir:` | lokales Verzeichnis im Skopeo-Verzeichnisformat | `dir:/tmp/app` |
| `oci:` | lokales OCI Image Layout | `oci:/tmp/app:1.0` |
| `oci-archive:` | OCI-Archivdatei | `oci-archive:/tmp/app.tar:1.0` |
| `docker-archive:` | Docker-kompatibles Archiv | `docker-archive:/tmp/app.tar:app:1.0` |
| `containers-storage:` | lokaler Podman/Buildah-Image-Storage | `containers-storage:localhost/app:1.0` |

Für Registry-Zugriffe ist fast immer `docker://` erforderlich:

```bash
skopeo inspect docker://registry.example.com/team/app:1.0
```

---

## 3. Installation und Hilfe

### RHEL, CentOS Stream oder Fedora

```bash
sudo dnf install skopeo
```

### Version anzeigen

```bash
skopeo --version
```

### Verfügbare Unterbefehle anzeigen

```bash
skopeo --help
```

### Hilfe zu einem Unterbefehl

```bash
skopeo inspect --help
skopeo copy --help
skopeo sync --help
```

---

## 4. Images untersuchen

### Metadaten eines Remote-Images anzeigen

```bash
skopeo inspect docker://docker.io/library/nginx:latest
```

Die Ausgabe enthält unter anderem:

- Image-Name
- Digest
- Tags
- Erstellungszeitpunkt
- Architektur
- Betriebssystem
- Labels
- Environment-Variablen
- konfigurierte Startparameter

### Nur den Digest anzeigen

```bash
skopeo inspect \
  --format '{{.Digest}}' \
  docker://docker.io/library/nginx:latest
```

### Architektur anzeigen

```bash
skopeo inspect \
  --format '{{.Architecture}}' \
  docker://docker.io/library/nginx:latest
```

### Betriebssystem und Architektur anzeigen

```bash
skopeo inspect \
  --format '{{.Os}}/{{.Architecture}}' \
  docker://docker.io/library/nginx:latest
```

### Labels anzeigen

```bash
skopeo inspect \
  --format '{{json .Labels}}' \
  docker://docker.io/library/nginx:latest | jq
```

### Rohes Manifest anzeigen

```bash
skopeo inspect --raw \
  docker://docker.io/library/nginx:latest | jq
```

`--raw` ist besonders hilfreich, um Manifest Lists beziehungsweise OCI Image Indexes für Multi-Architektur-Images zu untersuchen.

### Image für eine bestimmte Architektur untersuchen

```bash
skopeo inspect \
  --override-os linux \
  --override-arch arm64 \
  docker://docker.io/library/nginx:latest
```

---

## 5. Tags eines Repositorys auflisten

Bei `list-tags` wird das **Repository ohne konkreten Tag** angegeben:

```bash
skopeo list-tags docker://docker.io/library/nginx
```

Beispielausgabe:

```json
{
  "Repository": "docker.io/library/nginx",
  "Tags": [
    "1.26",
    "1.27",
    "alpine",
    "latest"
  ]
}
```

### Nur die Tags ausgeben

```bash
skopeo list-tags docker://docker.io/library/nginx \
  | jq -r '.Tags[]'
```

### Tags alphabetisch sortieren

```bash
skopeo list-tags docker://docker.io/library/nginx \
  | jq -r '.Tags[]' \
  | sort
```

### Nach Tags suchen

```bash
skopeo list-tags docker://docker.io/library/nginx \
  | jq -r '.Tags[]' \
  | grep '^1\.27'
```

### Wichtig bei OpenShift-Registries

Der Repository-Pfad besteht normalerweise aus:

```text
<registry>/<namespace>/<image>
```

Beispiel:

```bash
skopeo list-tags \
  docker://registry.ocp4.example.com:8443/etherpad/etherpad
```

Dabei ist:

```text
registry.ocp4.example.com:8443   Registry
etherpad                         OpenShift-Projekt/Namespace
etherpad                         Repository beziehungsweise ImageStream
```

---

## 6. An einer Registry anmelden

### Interaktive Anmeldung

```bash
skopeo login registry.example.com
```

### Benutzername explizit angeben

```bash
skopeo login \
  --username myuser \
  registry.example.com
```

Das Passwort wird interaktiv abgefragt.

### Passwort über STDIN übergeben

```bash
printf '%s' "$REGISTRY_PASSWORD" \
  | skopeo login \
      --username "$REGISTRY_USER" \
      --password-stdin \
      registry.example.com
```

Diese Variante ist für Skripte und CI/CD sicherer als ein Passwort direkt in der Kommandozeile.

### Anmeldung prüfen

```bash
skopeo login --get-login registry.example.com
```

### Abmelden

```bash
skopeo logout registry.example.com
```

### Eigenes Authfile verwenden

```bash
skopeo login \
  --authfile ./auth.json \
  registry.example.com
```

Danach kann es für weitere Befehle verwendet werden:

```bash
skopeo inspect \
  --authfile ./auth.json \
  docker://registry.example.com/team/app:1.0
```

> Das Authfile enthält Registry-Anmeldedaten und sollte nicht in Git eingecheckt werden.

---

## 7. Images kopieren

### Image zwischen zwei Registries kopieren

```bash
skopeo copy \
  docker://quay.io/example/app:1.0 \
  docker://registry.example.com/team/app:1.0
```

Skopeo überträgt das Image direkt von Quelle zu Ziel. Ein vorheriges `pull` und anschließendes `push` ist nicht notwendig.

### Image unter einem anderen Tag kopieren

```bash
skopeo copy \
  docker://registry.example.com/team/app:1.0 \
  docker://registry.example.com/team/app:stable
```

Das kann zum Erzeugen eines zusätzlichen Tags verwendet werden.

### Unterschiedliche Zugangsdaten für Quelle und Ziel

```bash
skopeo copy \
  --src-creds "$SRC_USER:$SRC_PASSWORD" \
  --dest-creds "$DEST_USER:$DEST_PASSWORD" \
  docker://source.example.com/team/app:1.0 \
  docker://target.example.com/team/app:1.0
```

Für produktive Skripte sind getrennte Authfiles oder Secret-Mechanismen meist besser als Klartext-Credentials in Prozessargumenten.

### Quell- und Ziel-Authfile verwenden

```bash
skopeo copy \
  --src-authfile ./source-auth.json \
  --dest-authfile ./target-auth.json \
  docker://source.example.com/team/app:1.0 \
  docker://target.example.com/team/app:1.0
```

### Alle Architekturen eines Multi-Arch-Images kopieren

```bash
skopeo copy --all \
  docker://docker.io/library/nginx:latest \
  docker://registry.example.com/mirror/nginx:latest
```

Ohne `--all` wird normalerweise nur das zur aktuellen beziehungsweise vorgegebenen Plattform passende Image kopiert. Mit `--all` werden Manifest List beziehungsweise Image Index und alle referenzierten Plattform-Images übertragen.

### Bestimmte Architektur kopieren

```bash
skopeo copy \
  --override-os linux \
  --override-arch amd64 \
  docker://docker.io/library/nginx:latest \
  docker://registry.example.com/mirror/nginx:amd64
```

### Digest beim Kopieren beibehalten

```bash
skopeo copy \
  --preserve-digests \
  docker://source.example.com/team/app:1.0 \
  docker://target.example.com/team/app:1.0
```

Der Befehl schlägt fehl, wenn der Digest aufgrund einer notwendigen Konvertierung nicht beibehalten werden kann.

---

## 8. Images lokal sichern und übertragen

### Image in ein lokales Verzeichnis kopieren

```bash
skopeo copy \
  docker://docker.io/library/alpine:latest \
  dir:/tmp/alpine
```

### Image aus dem Verzeichnis in eine Registry kopieren

```bash
skopeo copy \
  dir:/tmp/alpine \
  docker://registry.example.com/mirror/alpine:latest
```

### OCI-Archiv erstellen

```bash
skopeo copy \
  docker://docker.io/library/alpine:latest \
  oci-archive:/tmp/alpine-oci.tar:latest
```

### OCI-Archiv in eine Registry übertragen

```bash
skopeo copy \
  oci-archive:/tmp/alpine-oci.tar:latest \
  docker://registry.example.com/mirror/alpine:latest
```

### Docker-Archiv erstellen

```bash
skopeo copy \
  docker://docker.io/library/alpine:latest \
  docker-archive:/tmp/alpine.tar:alpine:latest
```

### Image in den lokalen Podman-Storage kopieren

```bash
skopeo copy \
  docker://docker.io/library/alpine:latest \
  containers-storage:localhost/alpine:latest
```

Danach erscheint das Image beispielsweise bei:

```bash
podman images
```

### Lokales Podman-Image in eine Registry kopieren

```bash
skopeo copy \
  containers-storage:localhost/alpine:latest \
  docker://registry.example.com/team/alpine:latest
```

> Root und Rootless Podman verwenden unterschiedliche lokale Container-Storages. Führe Skopeo daher möglichst mit demselben Benutzer aus wie Podman.

---

## 9. Repositorys synchronisieren

`skopeo sync` eignet sich zum Spiegeln mehrerer Tags oder Repositorys.

### Alle Tags eines Repositorys in ein lokales Verzeichnis synchronisieren

```bash
skopeo sync \
  --src docker \
  --dest dir \
  docker.io/library/alpine \
  /tmp/images
```

Wird bei einer Registry-Quelle kein Tag angegeben, synchronisiert Skopeo alle gefundenen Tags.

### Repository in eine andere Registry synchronisieren

```bash
skopeo sync \
  --src docker \
  --dest docker \
  docker.io/library/alpine \
  registry.example.com/mirror
```

### Synchronisation anhand einer YAML-Datei

Beispiel `images.yaml`:

```yaml
docker.io:
  images:
    library/alpine:
      - "3.20"
      - "latest"
    library/nginx:
      - "1.27"
```

Synchronisation:

```bash
skopeo sync \
  --src yaml \
  --dest docker \
  images.yaml \
  registry.example.com/mirror
```

Die exakte Zielstruktur hängt von Quelle, Ziel und verwendeten Sync-Optionen ab. Vor produktiven Spiegelungen empfiehlt sich ein Test mit einer separaten Ziel-Namespace.

---

## 10. Images löschen

### Image beziehungsweise Tag aus einer Registry löschen

```bash
skopeo delete \
  docker://registry.example.com/team/app:old
```

Wichtige Hinweise:

- Die Registry muss die Delete-API unterstützen.
- Skopeo markiert das Image beziehungsweise Manifest zur Löschung.
- Die tatsächliche Speicherbereinigung kann erst später durch die Garbage Collection der Registry erfolgen.
- Das Löschen eines gemeinsam verwendeten Manifests kann Auswirkungen auf mehrere Tags haben. Vorher Digest und Tag-Zuordnung prüfen.

---

## 11. OpenShift-Praxis

### Repository-Pfad eines ImageStreams anzeigen

```bash
oc get imagestreams
```

Kurzform:

```bash
oc get is
```

Beispiel:

```text
NAME       IMAGE REPOSITORY
etherpad   registry.ocp4.example.com:8443/etherpad/etherpad
```

Der vollständige Aufbau ist:

```text
<registry>/<namespace>/<imagestream>
```

### ImageStreams in allen Projekten suchen

```bash
oc get is -A
```

Gezielt nach einem Image suchen:

```bash
oc get is -A | grep etherpad
```

### Repository-URL direkt auslesen

```bash
oc get is etherpad \
  -o jsonpath='{.status.dockerImageRepository}{"\n"}'
```

### Tags eines OpenShift-Repositorys anzeigen

```bash
skopeo list-tags \
  docker://registry.ocp4.example.com:8443/etherpad/etherpad
```

### Mit aktuellem OpenShift-Token anmelden

```bash
skopeo login \
  --username "$(oc whoami)" \
  --password "$(oc whoami -t)" \
  registry.ocp4.example.com:8443
```

Sicherer über STDIN:

```bash
oc whoami -t \
  | skopeo login \
      --username "$(oc whoami)" \
      --password-stdin \
      registry.ocp4.example.com:8443
```

### Image in ein OpenShift-Projekt kopieren

```bash
skopeo copy \
  docker://docker.io/library/nginx:latest \
  docker://registry.ocp4.example.com:8443/myproject/nginx:latest
```

Dafür benötigt der angemeldete Benutzer Push-Berechtigungen im Zielprojekt.

### OpenShift-Registry mit unbekanntem Zertifikat

Nur zum Testen oder in kontrollierten Laborumgebungen:

```bash
skopeo inspect \
  --tls-verify=false \
  docker://registry.ocp4.example.com:8443/etherpad/etherpad:latest
```

Beim Kopieren können Quelle und Ziel getrennt behandelt werden:

```bash
skopeo copy \
  --src-tls-verify=false \
  --dest-tls-verify=false \
  docker://source.example.com/team/app:1.0 \
  docker://registry.ocp4.example.com:8443/myproject/app:1.0
```

Produktiv sollte die ausstellende CA stattdessen korrekt als vertrauenswürdig eingerichtet werden.

---

## 12. TLS, Zertifikate und Authentifizierung

### TLS-Prüfung für einen Registry-Zugriff deaktivieren

```bash
skopeo inspect \
  --tls-verify=false \
  docker://registry.example.com/team/app:1.0
```

### Nur TLS-Prüfung der Quelle deaktivieren

```bash
skopeo copy \
  --src-tls-verify=false \
  docker://source.example.com/team/app:1.0 \
  docker://target.example.com/team/app:1.0
```

### Nur TLS-Prüfung des Ziels deaktivieren

```bash
skopeo copy \
  --dest-tls-verify=false \
  docker://source.example.com/team/app:1.0 \
  docker://target.example.com/team/app:1.0
```

> `--tls-verify=false` sollte nicht die dauerhafte Lösung für produktive Systeme sein. Besser ist es, die interne CA korrekt zu verteilen.

### Typischer Speicherort der Anmeldedaten

Container-Tools wie Skopeo, Podman und Buildah verwenden Registry-Authentifizierungsdateien im Format `auth.json`. Der konkrete Standardpfad hängt unter anderem von Distribution, Benutzerkontext und Umgebungsvariablen ab.

Den gewünschten Pfad kann man explizit festlegen:

```bash
skopeo login \
  --authfile "$HOME/.config/containers/auth.json" \
  registry.example.com
```

---

## 13. Nützliche Ausgaben mit jq

### Digest ausgeben

```bash
skopeo inspect docker://docker.io/library/nginx:latest \
  | jq -r '.Digest'
```

### Erstellungszeitpunkt ausgeben

```bash
skopeo inspect docker://docker.io/library/nginx:latest \
  | jq -r '.Created'
```

### Architektur und Betriebssystem ausgeben

```bash
skopeo inspect docker://docker.io/library/nginx:latest \
  | jq -r '"\(.Os)/\(.Architecture)"'
```

### Environment-Variablen anzeigen

```bash
skopeo inspect docker://docker.io/library/nginx:latest \
  | jq -r '.Env[]'
```

### Alle Tags anzeigen

```bash
skopeo list-tags docker://docker.io/library/nginx \
  | jq -r '.Tags[]'
```

### Prüfen, ob ein bestimmter Tag existiert

```bash
TAG="1.27"

skopeo list-tags docker://docker.io/library/nginx \
  | jq -e --arg tag "$TAG" '.Tags | index($tag) != null'
```

Exit-Code:

- `0`: Tag wurde gefunden
- ungleich `0`: Tag wurde nicht gefunden

### Digest zweier Registries vergleichen

```bash
SOURCE_DIGEST="$(
  skopeo inspect \
    --format '{{.Digest}}' \
    docker://source.example.com/team/app:1.0
)"

TARGET_DIGEST="$(
  skopeo inspect \
    --format '{{.Digest}}' \
    docker://target.example.com/team/app:1.0
)"

if [[ "$SOURCE_DIGEST" == "$TARGET_DIGEST" ]]; then
  echo "Digests stimmen überein."
else
  echo "Digests unterscheiden sich."
fi
```

---

## 14. Häufige Fehler

### `repository not found`

Beispiel:

```text
repository not found
```

Mögliche Ursachen:

1. Namespace fehlt.
2. Repository-Name ist falsch.
3. Das Repository existiert nicht.
4. Die Registry verschleiert aus Sicherheitsgründen fehlende Berechtigungen als „not found“.

Falsch:

```bash
skopeo list-tags \
  docker://registry.ocp4.example.com:8443/etherpad
```

Richtig:

```bash
skopeo list-tags \
  docker://registry.ocp4.example.com:8443/etherpad/etherpad
```

In OpenShift den Pfad prüfen:

```bash
oc get is -A
```

---

### `unauthorized` oder `authentication required`

An der Registry anmelden:

```bash
skopeo login registry.example.com
```

Bei OpenShift:

```bash
oc whoami -t \
  | skopeo login \
      --username "$(oc whoami)" \
      --password-stdin \
      registry.ocp4.example.com:8443
```

Zusätzlich die Berechtigungen im Projekt prüfen:

```bash
oc auth can-i get imagestreams -n myproject
oc auth can-i update imagestreams/layers -n myproject
```

---

### `x509: certificate signed by unknown authority`

Die Registry verwendet ein Zertifikat, dessen CA lokal nicht vertraut wird.

Kurzfristiger Test:

```bash
skopeo inspect \
  --tls-verify=false \
  docker://registry.example.com/team/app:1.0
```

Dauerhafte Lösung:

- korrekte CA installieren
- Registry-Zertifikatskette prüfen
- Registry-Vertrauen der Container-Tools konfigurieren

---

### `manifest unknown`

Der angegebene Tag oder Digest existiert nicht:

```bash
skopeo list-tags docker://registry.example.com/team/app
```

Danach mit einem vorhandenen Tag prüfen:

```bash
skopeo inspect docker://registry.example.com/team/app:existing-tag
```

---

### `name unknown`

Meist ist der Repository-Pfad falsch oder das Repository existiert nicht. Namespace, Repository und Schreibweise prüfen.

---

### Zugriff auf öffentliche Registry schlägt wegen Rate Limit fehl

Mögliche Lösungen:

- an der Quell-Registry anmelden
- internes Registry-Mirror verwenden
- Zugriffe cachen beziehungsweise Images spiegeln
- unnötig häufige `inspect`- oder `list-tags`-Aufrufe vermeiden

---

## 15. Skopeo, Podman und Buildah

| Werkzeug | Hauptaufgabe |
|---|---|
| **Skopeo** | Remote-Images untersuchen, kopieren, synchronisieren und löschen |
| **Podman** | Container und Pods ausführen und verwalten |
| **Buildah** | Container-Images bauen und verändern |

Typischer Workflow:

```text
Buildah baut das Image
        ↓
Skopeo kopiert oder spiegelt das Image
        ↓
Podman beziehungsweise OpenShift führt es aus
```

Beispiel:

```bash
buildah bud -t localhost/example-app:1.0 .

skopeo copy \
  containers-storage:localhost/example-app:1.0 \
  docker://registry.example.com/team/example-app:1.0
```

---

## 16. Spickzettel

```bash
# Version
skopeo --version

# Remote-Image untersuchen
skopeo inspect docker://REGISTRY/NAMESPACE/IMAGE:TAG

# Rohes Manifest anzeigen
skopeo inspect --raw docker://REGISTRY/NAMESPACE/IMAGE:TAG | jq

# Digest anzeigen
skopeo inspect \
  --format '{{.Digest}}' \
  docker://REGISTRY/NAMESPACE/IMAGE:TAG

# Tags eines Repositorys anzeigen
skopeo list-tags docker://REGISTRY/NAMESPACE/IMAGE

# Nur Tag-Namen ausgeben
skopeo list-tags docker://REGISTRY/NAMESPACE/IMAGE \
  | jq -r '.Tags[]'

# Registry-Login
skopeo login REGISTRY

# Registry-Logout
skopeo logout REGISTRY

# Image zwischen Registries kopieren
skopeo copy \
  docker://SOURCE/NAMESPACE/IMAGE:TAG \
  docker://TARGET/NAMESPACE/IMAGE:TAG

# Multi-Arch-Image vollständig kopieren
skopeo copy --all \
  docker://SOURCE/NAMESPACE/IMAGE:TAG \
  docker://TARGET/NAMESPACE/IMAGE:TAG

# Image in ein Verzeichnis sichern
skopeo copy \
  docker://REGISTRY/NAMESPACE/IMAGE:TAG \
  dir:/tmp/image

# OCI-Archiv erstellen
skopeo copy \
  docker://REGISTRY/NAMESPACE/IMAGE:TAG \
  oci-archive:/tmp/image.tar:TAG

# Image löschen
skopeo delete docker://REGISTRY/NAMESPACE/IMAGE:TAG

# OpenShift: ImageStreams anzeigen
oc get is -A

# OpenShift: exakten Repository-Pfad ausgeben
oc get is IMAGESTREAM -n NAMESPACE \
  -o jsonpath='{.status.dockerImageRepository}{"\n"}'

# OpenShift: Registry-Login mit aktuellem Token
oc whoami -t \
  | skopeo login \
      --username "$(oc whoami)" \
      --password-stdin \
      REGISTRY
```

---

## Weiterführende Dokumentation

- Skopeo-Projekt: https://github.com/containers/skopeo
- Skopeo-Manpage: https://www.mankier.com/1/skopeo
- Container-Transports: https://www.mankier.com/5/containers-transports
- Registry-Authentifizierung: https://www.mankier.com/5/containers-auth.json
- Red Hat Container-Dokumentation: https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/building_running_and_managing_containers/

---

## Sicherheitshinweise

- Passwörter und Tokens nicht in Git speichern.
- In Skripten bevorzugt `--password-stdin`, Authfiles oder Secret Stores verwenden.
- `--tls-verify=false` nur für kontrollierte Tests einsetzen.
- Vor `skopeo delete` Tags, Digests und mögliche gemeinsame Manifest-Referenzen prüfen.
- Bei Multi-Arch-Images bewusst entscheiden, ob nur eine Plattform oder mit `--all` der vollständige Image Index kopiert werden soll.
