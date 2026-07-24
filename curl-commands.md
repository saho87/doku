# curl Referenz – OpenShift / Kubernetes / Linux

> Kompakte Referenz zum Debuggen von HTTP, HTTPS, TLS, APIs und OpenShift-Routes.

## Inhaltsverzeichnis

- [1. Grundlagen](#1-grundlagen)
- [2. HTTP Requests](#2-http-requests)
- [3. HTTP Header](#3-http-header)
- [4. Verbose & Debugging](#4-verbose--debugging)
- [5. HTTPS & TLS](#5-https--tls)
- [6. Redirects](#6-redirects)
- [7. Authentifizierung](#7-authentifizierung)
- [8. Daten senden](#8-daten-senden)
- [9. Dateien herunterladen](#9-dateien-herunterladen)
- [10. APIs testen](#10-apis-testen)
- [11. OpenShift Debugging](#11-openshift-debugging)
- [12. Troubleshooting](#12-troubleshooting)
- [13. Cheatsheet](#13-cheatsheet)

---

# 1. Grundlagen

| Option | Bedeutung |
|---|---|
| curl URL | HTTP GET Request |
| -I | HEAD Request |
| -v | Verbose |
| -k | Zertifikatsprüfung deaktivieren |
| -L | Redirects folgen |
| -H | HTTP Header setzen |
| -X | HTTP-Methode wählen |

```text
Client
   |
 curl
   |
HTTP / HTTPS
   |
Server
```

💡 **Merke:** `curl` verwendet standardmäßig **GET**.

---

# 2. HTTP Requests

GET

```bash
curl http://service
```

HEAD

```bash
curl -I http://service
```

Nur HTTP-Status

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://service
```

...
