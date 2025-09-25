# Netzwerk-Fehleranalyse im Linux-Umfeld

Diese Dokumentation beschreibt, wie man mit typischen Linux-Netzwerktools (z. B. `ping`, `curl`, `nslookup`) systematisch Probleme bei der Erreichbarkeit einer Applikation analysieren und beheben kann.

---

## 1. Grundlegendes Vorgehen

1. **Hostname/IP auflösen** – Funktioniert die DNS-Auflösung?
2. **Netzwerkverbindung prüfen** – Ist das Ziel grundsätzlich erreichbar?
3. **Port-Erreichbarkeit testen** – Ist der richtige Dienst/Port offen?
4. **Protokoll-Ebene prüfen** – Antwortet die App korrekt (HTTP, TCP, etc.)?
5. **Fehler eingrenzen** – Liegt das Problem an DNS, Netzwerk, Firewall, oder in der App?

---

## 2. Tools und Beispiele

### 2.1 `ping`
Prüft die grundsätzliche Erreichbarkeit eines Hosts auf ICMP-Ebene.
Nutzt ICMP Schicht 3, keine Ports notwendig

```bash
ping example.com
```
👉 Erwartung: Ausgabe von Antwortzeiten. Falls keine Antwort: DNS-Problem oder Host nicht erreichbar.

---

### 2.2 `nslookup` / `dig`
Prüft die DNS-Auflösung.

Hierbei kein Protokoll am Anfang und kein / am Ende angeben!

```bash
nslookup example.com
dig example.com A
```
👉 Erwartung: Ausgabe einer IP-Adresse. Falls leer oder Fehlermeldung: DNS-Problem.

---

### 2.3 `traceroute`
Zeigt den Weg der Pakete bis zum Ziel.
```bash
traceroute example.com
```
👉 Erwartung: Auflistung von Routern bis zum Ziel. Abbruch auf halbem Weg → Routingproblem oder Firewall.

---

### 2.4 `curl`
Prüft die Erreichbarkeit einer App auf HTTP-/HTTPS-Ebene.
```bash
# Einfacher Test
curl http://example.com

# Nur Header anzeigen
curl -I http://example.com

# Mit detaillierten Debug-Infos
curl -v http://example.com
```
👉 Erwartung: HTTP-Statuscode (z. B. `200 OK`). Fehler wie `Connection refused` oder `Timeout` weisen auf Port-/Firewall-Probleme hin.

---

### 2.5 `telnet` oder `nc` (netcat)
Prüfen, ob ein Port erreichbar ist.
```bash
# Mit telnet
telnet example.com 80

# Mit netcat
nc -zv example.com 80
```
👉 Erwartung: `succeeded` oder Verbindungsaufbau. Falls „refused“ oder Timeout → Port nicht offen oder blockiert.

---

### 2.6 `ss` / `netstat`
Lokale Ports und Verbindungen anzeigen.
```bash
ss -tulpen   # zeigt offene Ports
netstat -tulpen
```
👉 Erwartung: Die App lauscht auf der richtigen IP und dem richtigen Port.

---

### 2.7 `tcpdump`
Netzwerkverkehr mitschneiden, um tiefergehende Analysen durchzuführen.
```bash
sudo tcpdump -i eth0 host example.com and port 80
```
👉 Erwartung: Sichtbare Pakete beim Verbindungsaufbau. Falls keine → Traffic blockiert.

---

## 3. Beispielhafter Ablauf einer Analyse

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

## 4. Zusammenfassung
- **DNS** mit `nslookup`, `dig` prüfen.
- **Grundlegende Erreichbarkeit** mit `ping`.
- **Routing** mit `traceroute`.
- **Porttests** mit `nc`, `telnet`.
- **HTTP-Tests** mit `curl`.
- **Lokale Dienste** mit `ss`, `netstat`.
- **Traffic-Analyse** mit `tcpdump`.

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
