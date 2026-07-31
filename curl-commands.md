# curl Referenz

## Inhaltsverzeichnis

1. [Grundlagen](#1-grundlagen)
2. [Aufbau eines curl-Befehls](#2-aufbau-eines-curl-befehls)
3. [HTTP-Methoden](#3-http-methoden)
4. [Header](#4-header)
5. [HTTPS & TLS](#5-https--tls)
6. [Authentifizierung](#6-authentifizierung)
7. [Cookies](#7-cookies)
8. [Daten senden](#8-daten-senden)
9. [Downloads](#9-downloads)
10. [Redirects](#10-redirects)
11. [Debugging](#11-debugging)
12. [OpenShift](#12-openshift)
13. [REST APIs](#13-rest-apis)
14. [Fehleranalyse](#14-fehleranalyse)
15. [Cheatsheet](#15-cheatsheet)

---

# 1. Grundlagen

## Was ist curl?

`curl` (Client URL) ist ein Kommandozeilenwerkzeug zum Übertragen von Daten über verschiedenste Netzwerkprotokolle.

Unterstützte Protokolle (Auswahl):

- HTTP
- HTTPS
- FTP
- FTPS
- SFTP
- SCP
- LDAP
- MQTT
- SMB

Die häufigste Verwendung ist das Testen von

- Webservern
- REST APIs
- OpenShift Routes
- Kubernetes Services
- TLS-Verbindungen

---

## Aufbau

```bash
curl [OPTIONEN] URL
```

Beispiel

```bash
curl https://example.com
```

---

## Standardverhalten

Ohne Optionen führt curl einen **HTTP GET Request** aus.

```bash
curl https://example.com
```

entspricht

```http
GET / HTTP/1.1
Host: example.com
```

---

# 2. Aufbau eines curl-Befehls

## Allgemeine Syntax

```bash
curl [Optionen] URL
```

Beispiel

```bash
curl -v https://example.com
```

| Bestandteil | Bedeutung |
|-------------|-----------|
| curl | Programm |
| -v | Verbose |
| URL | Ziel |

---

## Mehrere Optionen kombinieren

```bash
curl -k -L -v https://example.com
```

oder

```bash
curl -kLv https://example.com
```

---

# 3. HTTP-Methoden

## GET (Standard)

```bash
curl https://example.com
```

---

## HEAD

Nur Header abrufen.

```bash
curl -I https://example.com
```

Ausgabe

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
```

---

## POST

```bash
curl \
-X POST \
https://example.com/api
```

---

## PUT

```bash
curl \
-X PUT \
https://example.com/api
```

---

## DELETE

```bash
curl \
-X DELETE \
https://example.com/api/1
```

---

## PATCH

```bash
curl \
-X PATCH \
https://example.com/api/1
```

---

# 4. Header

## Header anzeigen

```bash
curl -I https://example.com
```

---

## Verbose Header

```bash
curl -v https://example.com
```

Dabei werden angezeigt:

- Request Header
- Response Header

---

## Eigene Header setzen

```bash
curl \
-H "Accept: application/json" \
https://example.com
```

Mehrere Header

```bash
curl \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "X-Test: Demo" \
https://example.com
```

---

## Authorization Header

```bash
curl \
-H "Authorization: Bearer TOKEN" \
https://example.com
```

---

# 5. HTTPS & TLS

HTTPS verwenden

```bash
curl https://example.com
```

---

## TLS ignorieren

Selbstsignierte Zertifikate akzeptieren.

```bash
curl -k https://example.com
```

---

## TLS analysieren

```bash
curl -Iv https://example.com
```

Angezeigt werden u.a.

- TLS Version
- Cipher
- Zertifikat
- Issuer
- Subject
- Ablaufdatum

---

## Warum findet nur bei HTTPS ein TLS-Handshake statt?

HTTP

```text
Client
   │
TCP
   │
HTTP
```

HTTPS

```text
Client
   │
TCP
   │
TLS Handshake
   │
HTTP
```

Merke:

- HTTP → keine Verschlüsselung
- HTTPS → TLS wird aufgebaut

---

## Wichtige TLS-Optionen

| Option | Bedeutung |
|---------|-----------|
| -k | Zertifikat ignorieren |
| -I | Nur Header |
| -v | Verbose |
| -Iv | Header + TLS |
| --cacert | Eigenes CA Zertifikat |
| --cert | Client-Zertifikat |
| --key | Privater Schlüssel |

---

# 6. Authentifizierung

## Basic Auth

```bash
curl \
-u user:password \
https://example.com
```

---

## Bearer Token

```bash
curl \
-H "Authorization: Bearer TOKEN" \
https://example.com
```

---

## Client Zertifikat

```bash
curl \
--cert client.crt \
--key client.key \
https://example.com
```

---

## CA Zertifikat

```bash
# Prüfung: Wurde server.crt von dieser CA signiert?Ist die Signatur gültig?Ist das Zertifikat noch gültig (nicht abgelaufen)? Passt der Hostname ?(example.com) zum Zertifikat?
curl \
--cacert ca.crt \
https://example.com
```

---

# 7. Cookies

## Cookies speichern

```bash
curl -c cookies.txt https://example.com
```

Die vom Server gesetzten Cookies werden in der Datei gespeichert.

---

## Cookies verwenden

```bash
curl -b cookies.txt https://example.com
```

---

## Cookies direkt setzen

```bash
curl \
-b "SESSIONID=123456" \
https://example.com
```

---

# 8. Daten senden

## POST Formulardaten

```bash
curl \
-X POST \
-d "username=max&password=secret" \
https://example.com/login
```

---

## Mehrere Parameter

```bash
curl \
-d "name=Sascha" \
-d "city=Gotha" \
https://example.com
```

---

## JSON senden

```bash
curl \
-X POST \
-H "Content-Type: application/json" \
-d '{"name":"Sascha"}' \
https://example.com/api
```

---

## JSON aus Datei

```bash
curl \
-X POST \
-H "Content-Type: application/json" \
--data @payload.json \
https://example.com/api
```

---

## Datei als Request Body

```bash
curl \
--data-binary @payload.json \
https://example.com/api
```

---

## Multipart Upload

```bash
curl \
-F "file=@bild.png" \
https://example.com/upload
```

---

# 9. Downloads

## Datei herunterladen

```bash
curl -O https://example.com/image.iso
```

Dateiname bleibt erhalten.

---

## Eigenen Dateinamen vergeben

```bash
curl \
-o ubuntu.iso \
https://example.com/image.iso
```

---

## Download fortsetzen

```bash
curl \
-C - \
-O https://example.com/image.iso
```

---

## Mehrere Dateien herunterladen

```bash
curl -O https://server/file1
curl -O https://server/file2
curl -O https://server/file3
```

---

# 10. Redirects

## Redirect anzeigen

```bash
curl -I http://example.com
```

Beispiel

```http
HTTP/1.1 301 Moved Permanently
Location: https://example.com
```

---

## Redirect folgen

```bash
curl -L http://example.com
```

---

## Mehrere Redirects verfolgen

```bash
curl -Lv http://example.com
```

---

# 11. Debugging

## Verbose

```bash
curl -v https://example.com
```

Zeigt

- DNS
- TCP
- Header
- HTTP Status

---

## TLS analysieren

```bash
curl -Iv https://example.com
```

Zeigt zusätzlich

- Zertifikat
- TLS Version
- Cipher
- Ablaufdatum

---

## Maximale Details

```bash
curl --trace-ascii trace.log https://example.com
```

---

## HTTP Statuscode ausgeben

```bash
curl \
-s \
-o /dev/null \
-w "%{http_code}\n" \
https://example.com
```

---

## Response Time

```bash
curl \
-o /dev/null \
-s \
-w "%{time_total}\n" \
https://example.com
```

---

## Mehrere Werte ausgeben

```bash
curl \
-o /dev/null \
-s \
-w "
Status : %{http_code}
Zeit   : %{time_total}
IP     : %{remote_ip}
Bytes  : %{size_download}
\n" \
https://example.com
```

---

# 12. OpenShift

## Route testen

```bash
curl https://todo.apps.ocp.example.com
```

---

## Route mit Header

```bash
curl -I https://todo.apps.ocp.example.com
```

---

## TLS prüfen

```bash
curl -Iv https://todo.apps.ocp.example.com
```

---

## Edge Route

```bash
curl \
-kIv \
https://route.apps.cluster.example.com
```

---

## Service testen

```bash
curl http://todo-service:8080
```

---

## Pod testen

```bash
oc exec podname -- \
curl localhost:8080
```

---

## Kubernetes API

```bash
TOKEN=$(oc whoami -t)

curl \
-H "Authorization: Bearer $TOKEN" \
https://api.cluster.example.com:6443/api
```

---

## Typischer Debugging-Ablauf

```text
Browser
    │
Route
    │
Router
    │
Service
    │
Endpoints
    │
Pod
```

curl eignet sich hervorragend, um jede Ebene einzeln zu testen.

---

# 13. REST APIs

## GET

```bash
curl https://api.example.com/users
```

---

## POST

```bash
curl \
-X POST \
-H "Content-Type: application/json" \
-d '{"name":"Max"}' \
https://api.example.com/users
```

---

## PUT

```bash
curl \
-X PUT \
-H "Content-Type: application/json" \
-d '{"name":"Peter"}' \
https://api.example.com/users/5
```

---

## DELETE

```bash
curl \
-X DELETE \
https://api.example.com/users/5
```

---

# 14. Fehleranalyse

| Fehler | Ursache | Lösung |
|---------|----------|---------|
| Connection refused | Dienst läuft nicht | Pod oder Service prüfen |
| Timeout | Netzwerkproblem | Firewall, Route prüfen |
| 301 | Redirect | `curl -L` |
| 302 | Redirect | Redirect folgen |
| 401 | Authentifizierung fehlt | Token prüfen |
| 403 | Keine Berechtigung | RBAC prüfen |
| 404 | Falscher Pfad | Route oder URL prüfen |
| 500 | Serverfehler | Logs prüfen |
| 503 | Keine Endpoints | `oc get endpoints` |

---

## Typische OpenShift-Kommandos

```bash
oc get routes

oc get svc

oc get endpoints

oc logs POD

oc describe route NAME

oc exec POD -- curl localhost:8080
```

---

# 15. Cheatsheet

## Die wichtigsten Optionen

| Option | Beschreibung |
|---------|--------------|
| -I | HEAD Request |
| -L | Redirect folgen |
| -k | Zertifikat ignorieren |
| -v | Verbose |
| -s | Silent |
| -S | Fehler trotz Silent anzeigen |
| -o | Ausgabe in Datei |
| -O | Original-Dateiname |
| -H | HTTP Header |
| -d | Daten senden |
| -X | HTTP Methode |
| -u | Basic Auth |
| -F | Multipart Upload |
| -c | Cookies speichern |
| -b | Cookies verwenden |
| --cacert | CA Zertifikat |
| --cert | Client Zertifikat |
| --key | Privater Schlüssel |
| --trace-ascii | Vollständiger Trace |

---

## EX280 Merksätze

✅ `curl` verwendet standardmäßig **GET**

✅ `-I` = HEAD

✅ `-v` = HTTP Debugging

✅ `-Iv` = TLS + Header

✅ `-k` = Zertifikat ignorieren

✅ `-L` = Redirect folgen

✅ `-o` = eigener Dateiname

✅ `-O` = Original-Dateiname

✅ `-H` = Header setzen

✅ `-d` = Request Body senden

✅ `-F` = Datei hochladen

✅ `curl` ist das wichtigste Werkzeug zum Testen von:

- OpenShift Routes
- Kubernetes Services
- REST APIs
- TLS
- Zertifikaten
- HTTP Statuscodes

---

# Zusammenfassung

```text
GET                 curl URL
HEAD                curl -I URL
Verbose             curl -v URL
TLS Debug           curl -Iv URL
Ignore Cert         curl -k URL
Redirect            curl -L URL
POST JSON           curl -X POST -H "Content-Type: application/json" -d '{}'
Header              curl -H "Header: Wert"
Download            curl -O URL
Download Name       curl -o DATEI URL
Statuscode          curl -s -o /dev/null -w "%{http_code}"
Cookie speichern    curl -c cookies.txt
Cookie verwenden    curl -b cookies.txt
Basic Auth          curl -u user:pass
Bearer Token        curl -H "Authorization: Bearer TOKEN"
```
