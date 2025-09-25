# Nützliche PowerShell-Befehle

Eine Sammlung praktischer PowerShell-Befehle für Netzwerk, System, Dateien, Prozesse und allgemeine Aufgaben.  
Ideal für die Arbeit auf Windows-Servern oder -Clients.

---

## 🔌 Netzwerk & Verbindungen

| Befehl | Beschreibung |
|--------|--------------|
| `Test-NetConnection -ComputerName google.com -Port 443` | Prüft, ob ein Host über einen bestimmten Port erreichbar ist. ! über IP suchen, nicht FQDN|
| `Get-NetIPAddress` | Zeigt alle lokalen IP-Adressen. |
| `Get-NetTCPConnection` | Listet alle aktiven TCP-Verbindungen auf. |
| `Resolve-DnsName heise.de` | Führt eine DNS-Abfrage durch (besser als `nslookup`). |
| `Get-NetRoute` | Zeigt die aktuelle Routing-Tabelle. |

---

## 💻 System & Hardware

| Befehl | Beschreibung |
|--------|--------------|
| `Get-ComputerInfo` | Vollständige Hardware- und OS-Informationen. |
| `Get-WmiObject Win32_LogicalDisk` | Informationen zu Laufwerken, Speicherplatz und Dateisystem. |
| `Get-Process` | Liste aller laufenden Prozesse. |
| `Get-Service` | Listet alle Windows-Dienste. |
| `Get-EventLog -LogName System -Newest 20` | Zeigt die letzten 20 Einträge im System-Eventlog. |

---

## 📂 Dateien & Ordner

| Befehl | Beschreibung |
|--------|--------------|
| `Get-ChildItem -Recurse` | Durchsucht alle Unterordner (wie `dir /s`). |
| `Get-ChildItem -Filter *.log -Recurse` | Findet alle `.log`-Dateien. |
| `Get-Content datei.txt -Tail 20` | Zeigt die letzten 20 Zeilen einer Datei. |
| `Measure-Object -Line -Word -Character` | Zählt Zeilen, Wörter und Zeichen in einer Datei. |
| `Get-FileHash datei.iso` | Berechnet den Hashwert einer Datei (z. B. SHA256). |

---

## ⚙ Prozesse & Verwaltung

| Befehl | Beschreibung |
|--------|--------------|
| `Stop-Process -Name notepad` | Beendet einen Prozess. |
| `Start-Process notepad.exe` | Startet einen Prozess. |
| `Get-Process \| Sort-Object CPU -Descending` | Sortiert Prozesse nach CPU-Verbrauch. |
| `Restart-Service spooler` | Startet den Druckerspooler neu. |

---

## 🔍 Nützliches Allerlei

| Befehl | Beschreibung |
|--------|--------------|
| `Get-History` | Zeigt die letzten PowerShell-Befehle an. |
| `Get-Clipboard` | Zeigt den Inhalt der Zwischenablage. |
| `Set-Clipboard "Hallo Welt"` | Schreibt Text in die Zwischena
