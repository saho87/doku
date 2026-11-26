# PostgreSQL – Grundlagen & Befehlsreferenz

> [!NOTE]
> Eigene Doku zur Postgres-Schulung – PostgreSQL 17, Rocky Linux, PGDG-Pakete.

---

## Inhalt

1. [Aufbau einer Postgres-Instanz](#1-aufbau-einer-postgres-instanz)
2. [User/Rollen und Rechte](#2-userrollen-und-rechte)
3. [Server verwalten: initdb, pg_ctl, systemd](#3-server-verwalten-initdb-pg_ctl-systemd)
4. [Fehlersuche, wenn der Server nicht startet](#4-fehlersuche-wenn-der-server-nicht-startet)
5. [Anmelden mit psql](#5-anmelden-mit-psql)
6. [Skripte ausführen](#6-skripte-ausführen)
7. [Schemas und search_path](#7-schemas-und-search_path)
8. [Orientierung auf einer fremden Instanz](#8-orientierung-auf-einer-fremden-instanz)
9. [psql-Metabefehle](#9-psql-metabefehle)
10. [SQL-Grundlagen und Datentypen](#10-sql-grundlagen-und-datentypen)
11. [Tablespaces](#11-tablespaces)
12. [Lösungsweg Aufgaben 4.1 und 4.2](#12-lösungsweg-aufgaben-41-und-42)
13. [Postgres zum Ausprobieren per Docker](#13-postgres-zum-ausprobieren-per-docker)
14. [Backup mit pgBackRest](#14-backup-mit-pgbackrest)

---

## 1. Aufbau einer Postgres-Instanz

```text
Cluster (= eine Postgres-Instanz, ein Datenverzeichnis, ein Port)
│
├── Rollen/User          ← gelten für den GANZEN Cluster
├── Tablespaces          ← gelten für den GANZEN Cluster
│
├── Datenbank "demodb"
│   ├── Schema "public"
│   │   └── Tabellen, Indizes, Views, Funktionen …
│   └── Schema "shop"
│       └── …
└── Datenbank "andere_db"
    └── Schema "public" …
```

- **Cluster:** eine laufende Postgres-Instanz mit genau einem Datenverzeichnis (`$PGDATA`, hier `/var/lib/pgsql/17/data`) und einem Port (Standard 5432).
- **Datenbank:** klar voneinander getrennt. Eine Verbindung geht immer auf genau eine Datenbank.
- **Schema:** ein „Ordner“ innerhalb einer Datenbank. Jede Datenbank hat standardmäßig das Schema `public`.
- **Objekte:** Tabellen, Indizes, Views, Funktionen usw. liegen immer in einem Schema.

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 2. User/Rollen und Rechte

**User (in Postgres „Rollen“) gelten clusterweit.** Eine Rolle existiert genau einmal und gehört weder zu einer Datenbank noch zu einem Schema.

**Was sie darf, regeln Rechte auf jeder Ebene einzeln.** Ähnlich wie bei Linux: Der User existiert systemweit, die Rechte hängen am Verzeichnis bzw. an der Datei.

| Ebene | Wichtige Rechte | Vergleich Linux |
|---|---|---|
| Datenbank | `CONNECT` (verbinden), `CREATE` (Schemas anlegen) | Laufwerk mounten |
| Schema | `USAGE` (hineinschauen), `CREATE` (Objekte anlegen) | `x`/`w` auf ein Verzeichnis |
| Tabelle | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, … | `r`/`w` auf eine Datei |

Damit `demo` die Tabelle `shop.bestellung` lesen kann, braucht er **alle drei Ebenen**:

```sql
GRANT CONNECT ON DATABASE demodb TO demo;
GRANT USAGE   ON SCHEMA shop      TO demo;
GRANT SELECT  ON shop.bestellung  TO demo;
```

Weitere Punkte:

- **Owner:** Wer ein Objekt anlegt, besitzt es und darf damit alles.
- **Superuser** (z. B. `postgres`) dürfen alles, ähnlich wie root.

### Rechte-Kurzform lesen (`\l`, `\dn+`, `\dp`)

```text
demo=arwd/postgres
```

Rolle `demo` hat `a` (INSERT), `r` (SELECT), `w` (UPDATE), `d` (DELETE), vergeben von `postgres`.

| Kürzel | Recht |
|---|---|
| `r` | SELECT |
| `a` | INSERT |
| `w` | UPDATE |
| `d` | DELETE |
| `D` | TRUNCATE |
| `x` | REFERENCES |
| `t` | TRIGGER |
| `U` | USAGE |
| `C` | CREATE |
| `c` | CONNECT |
| `T` | TEMPORARY |

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 3. Server verwalten: initdb, pg_ctl, systemd

### Was die Low-Level-Werkzeuge tun

| Befehl | Aufgabe |
|---|---|
| `initdb` | Legt einen **neuen, leeren Cluster** an (Datenverzeichnis mit Systemtabellen und Konfiguration). Einmalig. |
| `pg_ctl start` / `stop` / `restart` | Startet bzw. stoppt den Server für ein Datenverzeichnis |
| `pg_ctl reload` | Lädt `postgresql.conf` / `pg_hba.conf` neu, ohne Neustart |
| `pg_ctl status` | Läuft der Server? |
| `pg_ctl promote` | Macht ein Replikat (Standby) zum Primary |

### Wer das normalerweise erledigt

- **Paketinstallation (RHEL/Rocky, PGDG):** systemd-Dienst `postgresql-17.service`. Den Cluster legt man per `postgresql-17-setup initdb` (root) oder direkt mit `initdb` (als postgres) an.
- **Docker-Image:** Das Entrypoint-Skript macht `initdb`, der Container *ist* der Server.
- **Kubernetes-Operator** (CloudNativePG, Crunchy PGO): macht Initialisierung, Start, Stopp und Failover selbst.

`initdb` und `pg_ctl` direkt braucht man zum Lernen, bei Eigenbau-Setups, für mehrere Cluster auf einem Host oder zur Fehlersuche.

### Befehle in der Schulungsumgebung (als OS-User postgres)

```bash
# Cluster anlegen (einmalig)
/usr/pgsql-17/bin/initdb -D /var/lib/pgsql/17/data

# Dienst steuern (per sudo erlaubt – immer vollen Pfad /bin/systemctl verwenden!)
sudo /bin/systemctl start   postgresql-17.service
sudo /bin/systemctl stop    postgresql-17.service
sudo /bin/systemctl restart postgresql-17.service
sudo /bin/systemctl reload  postgresql-17.service
systemctl status postgresql-17.service

# Von Hand (nur wenn NICHT über systemd betrieben!)
/usr/pgsql-17/bin/pg_ctl -D /var/lib/pgsql/17/data -l logfile start
/usr/pgsql-17/bin/pg_ctl -D /var/lib/pgsql/17/data stop
```

> [!IMPORTANT]
> Einen Server **entweder** mit `pg_ctl` **oder** über systemd betreiben, nie beides. systemd weiß nichts von einem von Hand gestarteten Server und umgekehrt.

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 4. Fehlersuche, wenn der Server nicht startet

### Vorgehen

1. `systemctl status postgresql-17.service`: zeigt nur, **dass** es scheitert.
2. Journal lesen (als root):
   ```bash
   journalctl -u postgresql-17.service --no-pager -n 40
   journalctl -xeu postgresql-17.service
   ```
3. Steht dort `redirecting log output to logging collector process`, dann steht der eigentliche Fehler im **Postgres-Log**:
   ```bash
   systemctl stop postgresql-17.service          # Neustart-Schleife stoppen
   ls -lt /var/lib/pgsql/17/data/log/
   tail -30 /var/lib/pgsql/17/data/log/postgresql-*.log
   ```
   Gesucht ist die Zeile mit `FATAL:`.

### Typische Fehler

| Meldung | Ursache | Lösung |
|---|---|---|
| `"/var/lib/pgsql/17/data/" is missing or empty` | Cluster nie angelegt | `initdb -D /var/lib/pgsql/17/data` |
| `could not bind ... Address already in use` / `could not create any TCP/IP sockets` | Port 5432 belegt | Prozess finden und stoppen oder Port ändern |
| `syntax error in file ".../postgresql.conf"` / `invalid value for parameter` | Konfigurationsfehler | Konfiguration korrigieren |
| `Permission denied` / `could not create lock file` | Rechte oder SELinux | Owner `postgres`, Rechte `0700` aufs Datenverzeichnis |

### Port belegt: wer ist es?

```bash
ss -tlnp | grep 5432          # PID des Prozesses
ps -fp <PID>                  # Programm
systemctl status <PID>        # Dienst oder Session (session-…scope = von Hand gestartet)
ls -l /proc/<PID>/cwd         # Datenverzeichnis eines laufenden Postgres
```

Sauber stoppen:

```bash
/usr/pgsql-17/bin/pg_ctl -D <datenverzeichnis> stop   # als postgres
kill -INT <PID>                                       # Alternative: "fast shutdown"
```

Alternativ beide Instanzen parallel betreiben: in `postgresql.conf` `port = 5433` setzen.

> [!TIP]
> **Fall aus der Übung:** Ein früher von Hand mit `pg_ctl` gestarteter Server lief in einer verwaisten `su`-Session (`session-c2.scope`, `abandoned`) weiter und belegte Port 5432. Lösung: diesen Server stoppen, dann den systemd-Dienst starten.

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 5. Anmelden mit psql

| Weg | Befehl |
|---|---|
| Als aktueller OS-User auf gleichnamige DB | `psql` |
| Lokal über Unix-Socket | `psql -d demodb -U demo` |
| Über TCP/IP | `psql -h localhost -p 5432 -d demodb -U demo` |
| Verbindungs-URI | `psql "postgresql://demo:demo@localhost:5432/demodb"` |
| Schlüssel-Wert-String | `psql "host=localhost dbname=demodb user=demo"` |
| Umgebungsvariablen | `export PGDATABASE=demodb PGUSER=demo PGHOST=localhost` → `psql` |
| In laufender Sitzung wechseln | `\c demodb demo` |

**Passwort:** interaktiv eingeben, `PGPASSWORD` oder `~/.pgpass` (Format `host:port:db:user:passwort`, Rechte `0600`).

> [!WARNING]
> `PGPASSWORD` ist für andere Prozesse und in der Shell-History sichtbar – nur für Tests verwenden.

### Verbindung prüfen

```sql
\conninfo
SELECT current_user, current_database(), inet_server_addr(), inet_server_port();
SELECT version();
```

Bei Socket-Verbindung zeigt `\conninfo` „via socket in /run/postgresql“, bei `-h localhost` „on host localhost“.

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 6. Skripte ausführen

| Befehl | Wo |
|---|---|
| `\i /pfad/datei.sql` | innerhalb von psql |
| `psql -d demodb -U demo -f /pfad/datei.sql` | Shell |
| `psql -d demodb -U demo < /pfad/datei.sql` | Shell (Umleitung) |
| `psql -c "SELECT 1;"` | einzelnen Befehl von der Shell |

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 7. Schemas und search_path

**`search_path` ist für Postgres das, was `$PATH` für die Shell ist.**

- Schemas entsprechen Verzeichnissen.
- Der `search_path` ist die Liste der Schemas, in denen Postgres sucht, wenn ein Tabellenname **ohne** Schema angegeben wird.

```sql
SHOW search_path;        -- Standard: "$user", public
```

`"$user"` ist ein Platzhalter für den angemeldeten Benutzernamen. Ein Schema, das so heißt wie der User, wird also automatisch gefunden.

### Objekte außerhalb des search_path

`\dt` und `SELECT * FROM tabelle` finden nur Objekte im search_path. Für ein Schema `shop` gibt es drei Wege:

```sql
-- 1. Vollen "Pfad" angeben
SELECT * FROM shop.bestellung;
\dt shop.*
\di shop.*

-- 2. Alle Schemas durchsuchen
\dt *.*

-- 3. search_path erweitern (gilt nur für die aktuelle Sitzung)
SET search_path TO shop, public;
\dt
```

Dauerhaft setzen (Beispiele):

```sql
ALTER ROLE demo SET search_path TO shop, public;          -- für einen User
ALTER DATABASE demodb SET search_path TO shop, public;    -- für eine Datenbank
```

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 8. Orientierung auf einer fremden Instanz

Von außen nach innen: Cluster → Datenbank → Schema → Tabelle.

### 1. Wo bin ich, wer bin ich?

```sql
\conninfo
SELECT version();
\du                     -- alle Rollen und ihre Attribute (Superuser usw.)
```

### 2. Was gibt es im Cluster?

```sql
\l+                     -- Datenbanken: Größe, Owner, Zugriffsrechte
\db+                    -- Tablespaces mit Pfad und Größe
```

### 3. In eine Datenbank hineinschauen

```sql
\c <datenbank>
\dn+                    -- Schemas mit Owner und Rechten
SHOW search_path;
\dt *.*                 -- alle Tabellen in allen Schemas
\dv *.*                 -- Views
\df *.*                 -- Funktionen
\dx                     -- installierte Erweiterungen
```

> [!NOTE]
> `pg_catalog` und `information_schema` sind System-Schemas und können ignoriert werden.

### 4. Größte Tabellen finden

```sql
SELECT schemaname, relname,
       pg_size_pretty(pg_total_relation_size(relid)) AS groesse,
       n_live_tup AS zeilen_ca
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 20;
```

### 5. Einzelne Tabelle im Detail

```sql
\d+ schema.tabelle      -- Spalten, Indizes, Fremdschlüssel, Größe
\dp schema.tabelle      -- Zugriffsrechte
```

### 6. Wer ist gerade verbunden?

```sql
SELECT usename, datname, client_addr, state, query
FROM pg_stat_activity
WHERE backend_type = 'client backend';
```

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 9. psql-Metabefehle

| Befehl | Zeigt |
|---|---|
| `\conninfo` | aktuelle Verbindung |
| `\l` / `\l+` | Datenbanken |
| `\c db user` | Datenbank/User wechseln |
| `\du` | Rollen/User |
| `\db` / `\db+` | Tablespaces |
| `\dn` / `\dn+` | Schemas |
| `\dt` / `\dt schema.*` / `\dt *.*` | Tabellen |
| `\di` / `\di schema.*` | Indizes |
| `\dv` | Views |
| `\df` | Funktionen |
| `\dx` | Erweiterungen |
| `\d objekt` / `\d+ objekt` | Details zu einem Objekt |
| `\dp objekt` | Zugriffsrechte |
| `\i datei.sql` | Skript ausführen |
| `\x` | erweiterte Anzeige an/aus (gut für breite Zeilen) |
| `\timing` | Laufzeit von Abfragen anzeigen |
| `\?` | Hilfe zu Metabefehlen |
| `\h BEFEHL` | Hilfe zu SQL-Befehlen, z. B. `\h CREATE TABLE` |
| `\q` | beenden |

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 10. SQL-Grundlagen und Datentypen

### Syntax

> [!CAUTION]
> Spaltenname und Typ werden durch ein **Leerzeichen** getrennt (kein `=`). Einen Typ `string` gibt es nicht.

```sql
CREATE TABLE test (a int, b text);

INSERT INTO test VALUES (1, 'hallo'), (2, 'welt');
SELECT * FROM test WHERE a > 1 ORDER BY a DESC;
UPDATE test SET b = 'neu' WHERE a = 1;
DELETE FROM test WHERE a = 2;

DROP TABLE test;
```

### Häufige Datentypen

| Typ | Für |
|---|---|
| `int` / `bigint` | ganze Zahlen |
| `numeric(10,2)` | exakte Dezimalzahlen (z. B. Geld) |
| `text` | Text beliebiger Länge |
| `varchar(50)` | Text mit Maximallänge |
| `boolean` | wahr/falsch |
| `date` / `timestamptz` | Datum / Zeitpunkt mit Zeitzone |
| `serial` / `int GENERATED ALWAYS AS IDENTITY` | automatisch hochzählende ID |

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 11. Tablespaces

Ein **Tablespace** ist ein benannter Verweis auf ein Verzeichnis im Dateisystem. Damit bestimmt man, **wo physisch** Tabellen und Indizes liegen. Die logische Struktur und die SQL-Abfragen ändern sich dadurch nicht.

> [!TIP]
> **Bild:** Die Ordner einer Firma (Tabellen) stehen normalerweise alle im Keller (`pg_default`). Ein Tablespace ist ein zusätzlicher Raum, z. B. „2. Stock, schnell“. Die Mitarbeiter fragen weiter nach „Ordner Kunden“. Nur Postgres weiß, in welchem Raum er steht.

### Eingebaute Tablespaces

- `pg_default`: `$PGDATA/base`
- `pg_global`: clusterweite Systemkataloge

### Befehle

```sql
-- Verzeichnis muss existieren, leer sein und postgres gehören
CREATE TABLESPACE fast_ssd LOCATION '/mnt/nvme/pgdata';

CREATE TABLE events (...) TABLESPACE fast_ssd;
CREATE INDEX idx_events_ts ON events(ts) TABLESPACE fast_ssd;
ALTER TABLE events SET TABLESPACE fast_ssd;   -- sperrt und kopiert die Tabelle
CREATE DATABASE app TABLESPACE fast_ssd;

-- Übersicht
SELECT spcname, pg_tablespace_location(oid),
       pg_size_pretty(pg_tablespace_size(oid))
FROM pg_tablespace;
```

### Wozu und was beachten

- **Nutzen:** Storage-Tiering (heiße Daten auf SSD), akuter Platzmangel, eigener Tablespace für temporäre Dateien (`temp_tablespaces`).
- **Kein** Quota-Mechanismus, **nicht** eigenständig sicherbar oder an andere Cluster anhängbar.
- Backup/Restore (z. B. pgBackRest) und Replikas müssen die Tablespace-Pfade kennen.
- In Kubernetes/OpenShift selten nötig. Wenn doch, braucht jeder Tablespace ein eigenes PVC (CloudNativePG: `spec.tablespaces`).

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 12. Lösungsweg Aufgaben 4.1 und 4.2

### Aufgabe 4.1

<details>
<summary>Lösung anzeigen</summary>

```bash
su - postgres
cd /postgres/seminarunterlagen/uebungen/kapitel4/
chmod u+x demo.sh
./demo.sh
```

```bash
psql -d demodb -U demo                      # Socket
psql -h localhost -d demodb -U demo         # TCP
```

```sql
\conninfo
\db+        -- Tablespaces
\l+         -- Datenbanken
\dt         -- Tabellen
\di         -- Indizes
```

</details>

### Aufgabe 4.2

<details>
<summary>Lösung anzeigen</summary>

```bash
psql -h localhost -d demodb -U demo         # Passwort: demo
```

```sql
\i /postgres/seminarunterlagen/uebungen/kapitel4/demo.sql
\dn+                          -- Schemas
SHOW search_path;
\dt <schema>.*                -- Tabellen des neuen Schemas
\di <schema>.*                -- Indizes des neuen Schemas
-- oder:
SET search_path TO <schema>, public;
\dt
\di
```

</details>

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 13. Postgres zum Ausprobieren per Docker

```bash
docker run -d --name pg \
  -e POSTGRES_PASSWORD=geheim \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17

docker exec -it pg psql -U postgres
```

Grafische Clients: **DBeaver** oder **pgAdmin** (`localhost:5432`, User `postgres`).

Lernquellen: offizielles Tutorial unter postgresql.org/docs, Übungen auf pgexercises.com.

<p align="right"><a href="#inhalt">↑ nach oben</a></p>

---

## 14. Backup mit pgBackRest

### Grundprinzip: WAL

Postgres schreibt jede Änderung zuerst ins **WAL** (Write-Ahead Log, Dateien in `pg_wal/`) und erst danach in die Tabellendateien. Das WAL ist ein lückenloses Protokoll aller Änderungen.

```text
Vollbackup (Sonntag)  +  alle WAL-Dateien seitdem  =  jeder beliebige Zeitpunkt danach
```

Das ist **Point-in-Time-Recovery (PITR)**: z. B. den Stand von Mittwoch 14:03 Uhr wiederherstellen, kurz bevor eine Tabelle gelöscht wurde.

### Was pgBackRest macht

1. **WAL-Archivierung:** Jede volle WAL-Datei wird sofort komprimiert ins Repository kopiert (über `archive_command`).
2. **Backups des Datenverzeichnisses:**

| Typ | Inhalt |
|---|---|
| `full` | alles |
| `diff` | Änderungen seit dem letzten **Full**-Backup |
| `incr` | Änderungen seit dem letzten **beliebigen** Backup (kleinster, schnellster Typ) |

Dazu kommen Restore (inkl. PITR), Aufbewahrung (Retention), Kompression, Verschlüsselung, Parallelität sowie S3/Azure als Ziel.

### Begriffe

| Begriff | Bedeutung | In der Übung |
|---|---|---|
| **Stanza** | Konfiguration für **einen** Postgres-Cluster (was wird wohin gesichert) | `pg-stanza` |
| **Repository** (`repo1`) | Ablageort der Backups und WAL-Dateien; mehrere möglich (`repo1`, `repo2`) | `/var/lib/pgbackrest` |
| **pg1-path** | Datenverzeichnis des Clusters | `/var/lib/pgsql/17/data` |

### Konfiguration

`/etc/pgbackrest/pgbackrest.conf`:

```ini
[global]
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2
log-level-console=info

[pg-stanza]
pg1-path=/var/lib/pgsql/17/data
```

`postgresql.conf` (danach Neustart nötig, weil `archive_mode` geändert wird):

```ini
archive_mode = on
archive_command = 'pgbackrest --stanza=pg-stanza archive-push %p'
```

### WAL-Dateinamen lesen

Jede WAL-Datei ist 16 MB groß. Der Name besteht aus drei Teilen à 8 Hex-Zeichen:

```text
00000001 00000000 0000000A
│        │        └─ Segment-Nummer (zählt hoch: …0A, …0B, …0C)
│        └────────── logische Log-Datei (springt nach Segment …FF um)
└─────────────────── Timeline (erhöht sich nach einem PITR-Restore)
```

> [!NOTE]
> Postgres **recycelt** alte WAL-Dateien und benennt sie für die Zukunft um. Deshalb können Dateien mit höherer Nummer ein älteres Datum haben.

### Aufbau des Repositorys

```text
/var/lib/pgbackrest/
├── archive/pg-stanza/
│   ├── archive.info                 ← Metadaten (Postgres-Version, System-ID)
│   └── 17-1/                        ← Postgres-Version 17, Archiv-Generation 1
│       └── 0000000100000000/        ← Timeline + Log-Datei
│           └── 00000001000000000000000A-<prüfsumme>.gz
└── backup/pg-stanza/                ← die eigentlichen Backups
```

### Befehle (als OS-User postgres)

```bash
# Einmalig: Stanza im Repository anlegen
pgbackrest --stanza=pg-stanza stanza-create

# Prüfen, ob Konfiguration und WAL-Archivierung funktionieren
pgbackrest --stanza=pg-stanza check

# Backups
pgbackrest --stanza=pg-stanza --type=full backup
pgbackrest --stanza=pg-stanza --type=diff backup
pgbackrest --stanza=pg-stanza --type=incr backup

# Übersicht über Backups und archivierten WAL-Bereich
pgbackrest --stanza=pg-stanza info
```

`check` erzwingt einen WAL-Wechsel und wartet, bis die Datei im Repository ankommt. Erfolgreich sieht das so aus:

```text
INFO: check repo1 configuration (primary)
INFO: check repo1 archive for WAL (primary)
INFO: WAL segment 00000001000000000000000A successfully archived to '…' on repo1
INFO: check command end: completed successfully
```

### Restore

> [!WARNING]
> Postgres muss vor einem Restore **gestoppt** sein.

```bash
sudo /bin/systemctl stop postgresql-17.service

# Auf den neuesten Stand zurückspielen
pgbackrest --stanza=pg-stanza --delta restore

# Point-in-Time-Restore
pgbackrest --stanza=pg-stanza --delta \
  --type=time --target="2026-11-26 09:30:00+01" \
  --target-action=promote restore

sudo /bin/systemctl start postgresql-17.service
```

- `--delta`: Nur abweichende Dateien werden zurückgeschrieben. Ohne `--delta` muss das Datenverzeichnis leer sein.
- `--target-action=promote`: Nach Erreichen des Zeitpunkts wird der Server wieder normal beschreibbar.
- Nach einem PITR startet eine **neue Timeline** (`00000002…`).

### Übungsablauf PITR

1. `pgbackrest info`: Ohne Backup steht dort „no valid backups“.
2. Erstes `full`-Backup erstellen, danach erneut `info`.
3. Daten ändern, **Uhrzeit notieren**, dann eine Tabelle löschen.
4. Server stoppen, PITR auf die notierte Uhrzeit, Server starten.
5. Prüfen: Die Tabelle ist wieder da, und die neuen WAL-Dateien beginnen mit `00000002`.

> [!TIP]
> Im Kubernetes-Kontext (z. B. Helm-Chart mit eigenem Postgres) macht pgBackRest genau dasselbe. Dort ruft nur der Operator bzw. ein CronJob die Befehle auf.
