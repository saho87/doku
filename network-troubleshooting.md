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
