# Netzwerk-Fehleranalyse – Systematisches Vorgehen mit Tools

Diese Übersicht zeigt die typischen Schritte zur Analyse von Netzwerkproblemen und stellt jeweils die passenden Linux-Tools mit Beispielen vor.

---

## 1. Grundlegendes Vorgehen

### 1. **Hostname/IP auflösen – Funktioniert die DNS-Auflösung?**
- **Tools:**
  - `nslookup app.test.de` → einfache DNS-Abfrage (kein Protokoll und kein / am Ende angeben
  - `dig <hostname>` → detaillierte DNS-Abfrage
- **Ziel:** Liefert eine IP-Adresse zurück? Falls nein → DNS-Problem.
- **Hinweis:** tools nutzen direkt Primären DNS-Resolver -> kein /etc/hosts !!!

---

### 2. **Netzwerkverbindung prüfen – Ist das Ziel grundsätzlich erreichbar?**
- **Tools:**
  - `ping 8.8.8.8` → ICMP-Test (Layer 3 -> keine Portseinige Server 
  - `traceroute example.com` → zeigt den Paketweg
  - `tracert example.com` → WIN Paketweg
- **Ziel:** Antwortzeiten sichtbar? Route erreichbar? Wenn nicht → Routingproblem oder Host nicht verfügbar.
- **Hinweis** nutzt /etc/hosts

---

### 3. **Port-Erreichbarkeit testen – Ist der richtige Dienst/Port offen?**
- **Tools:**
  - `nc -zv example.com 443` → Port-Check (baut nur einen nackten TCP-Handshake auf, kein TLS, kein SNI) -> geht nicht bei jedem Server
  - `telnet example.com 22` → einfacher Porttest (älteres Tool)
  - `Test-NetConnection -ComputerName example.com -Port 443` (Windows, besser IP verwenden)
- **Ziel:** Verbindungsaufbau möglich? Falls „refused“ oder Timeout → Port nicht erreichbar oder blockiert.

---

### 4. **Protokoll-Ebene prüfen – Antwortet die App korrekt (HTTP, TCP, etc.)?**
- **Tools:**
  - `curl` → HTTP/HTTPS testen
  - `openssl s_client -connect <host>:443` → TLS/SSL testen
- **Beispiel:**
  ```bash
  curl -I http://example.com
  curl -v https://example.com
  openssl s_client -connect example.com:443
  ```
- **Ziel:** Liefert die App eine Antwort (z. B. HTTP 200)? Falls nicht → Problem in App oder Protokollebene.

---

### 5. **Fehler eingrenzen – Liegt das Problem an DNS, Netzwerk, Firewall, oder in der App?**
- **Tools:**
  - `ss -tulpen` → lokale Ports prüfen
  - `netstat -tulpen` → offene Ports anzeigen (älteres Tool)
  - `tcpdump` → Traffic mitschneiden
- **Beispiel:**
  ```bash
  ss -tulpen | grep 8080
  tcpdump -i eth0 host example.com and port 80
  ```
- **Ziel:**
  - Lauscht die App auf dem richtigen Port?
  - Kommen Pakete an?
  - Wird die Antwort blockiert?

---

## Zusammenfassung
- **DNS prüfen:** `nslookup`, `dig`
- **Erreichbarkeit testen:** `ping`, `traceroute`
- **Ports testen:** `nc`, `telnet`
- **Protokoll testen:** `curl`, `openssl`
- **Lokal & Traffic prüfen:** `ss`, `netstat`, `tcpdump

## Beispielhafter Ablauf einer Analyse

**Problem:** Eine Web-App unter `http://myapp.local:8080` ist nicht erreichbar.

1. **DNS prüfen:**
   ```bash
   nslookup myapp.local
   ```
   → Keine IP zurückgegeben → DNS-Fehler. Lösung: DNS-Eintrag setzen oder `/etc/hosts` ergänzen.

2. **Erreichbarkeit prüfen:**
   ```bash
   ping 192.168.1.50
   ```
   → Host erreichbar.

3. **Port testen:**
   ```bash
   nc -zv 192.168.1.50 8080
   ```
   → Verbindung verweigert → App lauscht nicht oder Firewall blockt.

4. **Auf Server prüfen:**
   ```bash
   ss -tulpen | grep 8080
   ```
   → App läuft nicht. Lösung: Service starten.

5. **Erneut testen:**
   ```bash
   curl -I http://192.168.1.50:8080
   ```
   → `200 OK` – Problem behoben.

---


Diese systematische Vorgehensweise ermöglicht es, Netzwerkprobleme von der DNS-Auflösung bis zur Anwendungsebene effizient einzugrenzen.

# OSI-Modell – Übersicht der 7 Schichten

Das OSI-Modell beschreibt Netzwerkkommunikation in **7 Schichten (Layern)**. Es dient dazu, Probleme systematisch einzugrenzen.

---

## **Layer 1 – Physical (Bitübertragungsschicht)**
- **Aufgabe:** Übertragung von Bits über physische Medien (Kabel, Funk, Glasfaser).
- **Beispiele:** Ethernet-Kabel, WLAN-Signal, Glasfaser, Spannung.
- **Typische Probleme:** Kabelbruch, defekter Port, kein Link.
- **Linux-Tools:**
  - `ethtool eth0`
  - `ip link`

---

## **Layer 2 – Data Link (Sicherungsschicht)**
- **Aufgabe:** Frames, MAC-Adressen, direkte Verbindung zwischen zwei Geräten.
- **Beispiele:** Ethernet (802.3), Wi-Fi (802.11), ARP.
- **Typische Probleme:** MAC-Konflikte, ARP-Fehler, Switch-Probleme.
- **Linux-Tools:**
  - `arp -n`
  - `tcpdump -i eth0 arp`

---

## **Layer 3 – Network (Vermittlungsschicht)**
- **Aufgabe:** Routing, IP-Adressen, logische Adressierung.
- **Beispiele:** IPv4, IPv6, ICMP (`ping`).
- **Typische Probleme:** Routing-Fehler, Subnetz falsch, ICMP blockiert.
- **Linux-Tools:**
  - `ping <ziel>`
  - `traceroute <ziel>`
  - `ip route`

---

## **Layer 4 – Transport**
- **Aufgabe:** Ende-zu-Ende-Kommunikation, Ports, Zuverlässigkeit.
- **Beispiele:** TCP, UDP.
- **Typische Probleme:** Firewall blockt Port, Verbindung verweigert.
- **Linux-Tools:**
  - `nc -zv <host> <port>`
  - `ss -tulpen`

---

## **Layer 5 – Session**
- **Aufgabe:** Verwaltung von Sitzungen (Aufbau, Aufrechterhaltung, Abbau).
- **Beispiele:** SSH-Sitzung, Datenbankverbindung.
- **Typische Probleme:** Session-Timeout, fehlerhafter Reconnect.
- **Linux-Tools:**
  - `ssh user@host`
  - Logfiles prüfen

---

## **Layer 6 – Presentation (Darstellungsschicht)**
- **Aufgabe:** Datenformat, Kodierung, Verschlüsselung.
- **Beispiele:** TLS/SSL, JSON, XML, JPEG.
- **Typische Probleme:** Zertifikatsfehler, Encoding-Probleme.
- **Linux-Tools:**
  - `openssl s_client -connect host:443`
  - `jq`, `iconv`

---

## **Layer 7 – Application (Anwendungsschicht)**
- **Aufgabe:** Schnittstelle zu Anwendungen und Diensten.
- **Beispiele:** HTTP, HTTPS, FTP, SMTP, DNS, REST-API.
- **Typische Probleme:** App antwortet nicht, 500-Fehler, falscher Service.
- **Linux-Tools:**
  - `curl http://host`
  - `dig example.com`

---

# 🎯 Beispiel Troubleshooting (HTTP-Fehler)

1. **Layer 1–2:** Kabel eingesteckt? WLAN verbunden? (`ip link`)
2. **Layer 3:** Host erreichbar? (`ping host`)
3. **Layer 4:** Port offen? (`nc -zv host 80`)
4. **Layer 5:** Session aktiv? (z. B. SSH/Datenbank prüfen)
5. **Layer 6:** TLS korrekt? (`openssl s_client -connect host:443`)
6. **Layer 7:** Antwortet die App? (`curl -I https://host`)
