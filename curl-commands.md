curl Referenz – OpenShift / Kubernetes / Linux

Kompakte Referenz zum Debuggen von HTTP, HTTPS, TLS, APIs und OpenShift-Routes.

Inhaltsverzeichnis

1. Grundlagen

2. HTTP Requests

3. HTTP Header

4. Verbose & Debugging

5. HTTPS & TLS

6. Redirects

7. Authentifizierung

8. Daten senden

9. Downloads & Uploads

10. OpenShift Debugging

11. Troubleshooting

12. Cheatsheet

1. Grundlagen

Begriff

Bedeutung

HTTP

Unverschlüsseltes Hypertext Transfer Protocol

HTTPS

HTTP über TLS

GET

Standardmethode von curl

HEAD

Liefert nur HTTP-Header

Body

Inhalt der HTTP-Antwort

curl
 │
 ├── DNS Lookup
 │
 ├── TCP Verbindung
 │
 ├── TLS Handshake (nur HTTPS)
 │
 ├── HTTP Request
 │
 └── HTTP Response

💡 Merke: Gibt man kein Schema an, verwendet curl standardmäßig http://.

2. HTTP Requests

Standard-GET

curl http://service

HEAD-Request

curl -I http://service

HTTP-Methode auswählen

curl -X POST http://server/api
curl -X PUT http://server/api
curl -X DELETE http://server/api/1

Nur HTTP-Status ausgeben

curl -s -o /dev/null -w "%{http_code}\n" http://service

3. HTTP Header

Nur Header anzeigen

curl -I https://example.com

Eigene Header setzen

curl \
-H "Accept: application/json" \
-H "Authorization: Bearer TOKEN" \
https://api.example.com

Header und Body getrennt speichern

curl -D headers.txt -o body.txt https://example.com

4. Verbose & Debugging

Verbose-Ausgabe

curl -v http://service

Zeigt:

DNS-Auflösung

TCP-Verbindung

Request Header

Response Header

TLS analysieren

curl -Iv https://route.apps.example.com

Zeigt zusätzlich:

TLS-Version

Cipher Suite

Zertifikat

Servername

TLS-Handshake

5. HTTPS & TLS

HTTPS verwenden

curl https://route.apps.example.com

Selbstsignierte Zertifikate akzeptieren

curl -k https://route.apps.example.com

TLS vollständig analysieren

curl -kIv https://route.apps.example.com

⚠️ Merke: Ein TLS-Handshake findet nur bei https:// statt.

6. Redirects

Redirect anzeigen

curl -I http://route.apps.example.com

Redirect automatisch folgen

curl -L http://route.apps.example.com

Typische Ausgabe

HTTP/1.1 301 Moved Permanently
Location: https://route.apps.example.com

7. Authentifizierung

Basic Authentication

curl -u user:password https://api.example.com

Bearer Token

curl \
-H "Authorization: Bearer TOKEN" \
https://api.example.com

Cookies speichern

curl -c cookies.txt https://example.com

Cookies wiederverwenden

curl -b cookies.txt https://example.com

8. Daten senden

POST mit Formulardaten

curl -X POST \
-d "username=test&password=secret" \
http://server/login

POST mit JSON

curl -X POST \
-H "Content-Type: application/json" \
-d '{"name":"Sascha"}' \
http://server/api

Datei als Body senden

curl --data-binary @payload.json \
-H "Content-Type: application/json" \
http://server/api

9. Downloads & Uploads

Datei herunterladen

curl -O https://example.com/file.txt

Eigenen Dateinamen vergeben

curl -o image.png https://example.com/logo.png

Datei hochladen

curl -T backup.tar.gz ftp://server/

Multipart Upload

curl -F "file=@image.png" http://server/upload

10. OpenShift Debugging

Service testen

curl http://todo-service:8080

Route testen

curl -Iv https://todo.apps.ocp4.example.com

HTTP → HTTPS Redirect prüfen

curl -I http://todo.apps.ocp4.example.com

Nur Statuscode prüfen

curl -s -o /dev/null -w "%{http_code}\n" \
https://todo.apps.ocp4.example.com

Typischer Workflow

curl
   │
Route
   │
Service
   │
Endpoints
   │
Pod

11. Troubleshooting

Problem

Ursache

Lösung

Connection refused

Pod oder Service nicht erreichbar

oc get pods, oc get svc

404 Not Found

Falscher Pfad

Route oder API prüfen

503 Service Unavailable

Keine Endpoints

oc get endpoints

SSL certificate problem

Selbstsigniertes Zertifikat

curl -k

Timeout

Netzwerkproblem

Firewall, Service oder Route prüfen

301 Redirect

HTTP wird auf HTTPS umgeleitet

curl -L

12. Cheatsheet

# GET
curl URL

# HEAD
curl -I URL

# Verbose
curl -v URL

# TLS + Header
curl -Iv URL

# Zertifikat ignorieren
curl -k URL

# Redirect folgen
curl -L URL

# Header setzen
curl -H "Header: Wert" URL

# POST JSON
curl -X POST \
-H "Content-Type: application/json" \
-d '{}' URL

# HTTP-Status
curl -s -o /dev/null -w "%{http_code}\n" URL

# Download
curl -O URL

# Download unter anderem Namen
curl -o DATEI URL

EX280 Merksätze

curl verwendet standardmäßig GET.

-I sendet einen HEAD-Request.

-v zeigt DNS, TCP und HTTP-Details.

-Iv zeigt zusätzlich TLS-Handshake und Zertifikate.

-k ignoriert Zertifikatsfehler.

-L folgt HTTP-Redirects automatisch.

Für OpenShift-Routes gehören curl -I, curl -Iv, curl -k und curl -L zu den wichtigsten Debugging-Befehlen.
