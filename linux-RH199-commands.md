

Formatierung:
https://docs.github.com/de/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax

<details>
  <summary>Zusatzinformationen</summary>  
### Heading
	
  1. Foo
  2. Bar
     * Baz
     * Qux

</details>

# Kapitel 1: System Access und SSH

```bash
# Befehle:
ssh-keygen, ssh-copy-id, ssh-add, ssh

# wichtige Dateien/Ordner:
/home/user/.ssh/

# key-pair erstellen
ssh-keygen -f .ssh/key-with-pass
ssh-copy-id -i .ssh/key-with-pass.pub user@remotehost	# Ablage in /.ssh/authorized_keys

# SSH-Agent wird gestartet und  ENV, die vom Agenten gesetzt werden, in aktuellen Shell-Umgebung aktiviert
eval $(ssh-agent)
# Hinzufügen eines priv Keys (passphrase muss dann nicht mehr eingegeben werden)		
ssh-add .ssh/key-with-pass				

# SSH-Verbindung mit Debugging und speziellem Key
ssh -v -i .ssh/key-with-pass user@remotehost	
```
# Kapitel 2: Verwalten von Dateien über Befehlszeile
```bash
# Befehle:
ln, stat

# wichtige Dateien/Ordner (detaillierter in pdf):
/boot 		# Dateien für Startvorgang
/dev 		# Spezielle Gerätedateien
/etc 		# Systemspezifische Konfigurationsdateien
/home 		# Benutzerverzeichnis für reguläre Benutzer
/root 		# Benutzerverzeichnis für root Benutzer
/run 		# Laufzeizdaten für Prozesse
/tmp 		# temporärer Dateiablage, automatische Löschung
/usr 		# installierte SW, Bibliotheken, Dateien
/usr/bin 	# Benutzerbefehle
/usr/sbin 	# Befehle Systemadministration
/usr/local 	# lokal angepasste SW
/var 		# Systemspezifische variable Daten (Logs, DB)

# Hard- und Softlinks
ln file.txt /tmp/file-hlink.txt 	# Hardlink
ln -s file.txt /tmp/file-slink.txt 	# Softlink

# Globbing
[user@host glob]$ ls [ac]* 		# Anfang mit a oder c
able alpha cast charlie

[user@host glob]$ ls ???? 		# 4 Zeichen lang
able alpha cast easy echo

# Klammererweiterung
[user@host glob]$ mkdir RHEL{7..9}
RHEL7 RHEL8 RHEL9

[user@host glob]$ echo {Sunday,Monday,Tuesday,Wednesday}.log
Sunday.log Monday.log Tuesday.log Wednesday.log

[user@host glob]$ echo file{a{1,2},b,c}.txt
filea1.txt filea2.txt fileb.txt filec.txt

# Befehlssubstitution
[user@host glob]$ echo The time is $(date +%M) minutes past $(date +%l%p).
The time is 26 minutes past 11AM.

# Schützen von Argumenten vor Erweiterung
[user@host glob]$ echo \$HOME 				# nur für ein Wort
$HOME
[user@host glob]$ echo "Will variable $myhost evaluate to $(hostname -s)?"
Will variable host evaluate to host? 			# Auflösung der Variablen
[user@host glob]$ echo 'Will variable $myhost evaluate to $(hostname -s)?'
Will variable $myhost evaluate to $(hostname -s)? 	# keine Auflösung der Variablen

stat file			# Zeitstempel und Infos einer Datei anzeigen
```
# Kapitel 3: Verwalten lokaler Benutzer und Gruppen
```bash
# Befehle:
id, su -, sudo -i, sudo 
useradd, userdmod, userdel - r, passwd
groupadd, groupmod, groupdel, newgrp
chage, date

# wichtige Dateien/Ordner:
/etc/passwd # Infos zu Benutzerkonten
/etc/group # Infos zu Gruppen
/etc/login.defs # Standardwerte für UID, Passwort Gültigkeit, UMASK
/etc/sudoers.d/ # Ablage Files für User Berechtigungen
/etc/sudoers 
/etc/shadow # Passwörter
/var/log/secure # Protokollierung von sudo Befehlen

# Gruppenmitgliedschaft eines Benutzers anzeigen
id sascha

# Benutzer wechseln
su - user01 
su user01 	# Non-Login-Shell - nicht empfohlen
su - 		# auf root wechseln
sudo -i 	# Wechsel auf root mit userPW

# Erstellen, Löschen und Ändern von Benutzern
useradd user01
userdel -r user01 				# Löschen inklusive Benutzerdateien/Ordner
usermod -c "Benutzer01" user01 			# Kommentarte Benutzerkonto aktualisieren
passwd user01 					# Passwort vergeben
usermod -L -e 2022-08-14 user01			# Benutzer sperren

# Vergabe von root-Rechten an group01
echo "%group01 ALL=(ALL) ALL" >> /etc/sudoers.d/group01

# Erstellen, Löschen und Ändern von Gruppen
groupadd -g 10000 group01  			# mit bestimmter GID erstellen
groupadd -r group01 				# Systemgruppe erstellen
usermod -aG group01 user01			# neuer Sekundäre Gruppenzuweisung
groupdel group01
newgrp group01					# temporäre Änderung der Primären Gruppe (Shell-Sitzung)

# Ändern der Passwort-Richtlinien (m-Mindestalter, M-Höchstalter, W-Warnzeitraum, I-Inaktivitätszeitraum)
chage -l user01					# Anzeige PW Richtlinien des Users
chage -m 0 -M 90 -W 7 -I 14 user01
chage -E $(date -d "+30 days" +%F) user01	# Ablaufdatum auf +30 Tage setzen
chage -d 0 user01				# User muss PW bei nächster Anmeldung ändern

# nologin-Shell
usermod -s /sbin/nologin user01
su - user # Anmeldung hier nicht mehr möglich

```
# Kapitel 4: Steuern des Zugriffs auf Dateien
```bash
# Befehle:
chmod, chown, (chgrp)
umask

# wichtige Dateien/Ordner:
/etc/profile, /etc/profile.d, /etc/bashrc (zusätzlich im Home)


# Änderungen von Berechtigungen 
chmod go-rw document.pdf
chmod -R ugoa+rwx /home/user/myfolder 	# rekursiv nur für Verzeichnisse
chmod -R g+rwX demodir 			# Ausführungsberechtigung nur für Verzeichnisse setzen
chmod 750 sampledir 			# (X=1, W=2, R=4) RWX für User, WX für Gruppe und nichts für andere (X=1, W=2, R=4)

# Anderung der Benutzer- oder Gruppeneigentümer für Dateien und Verzeichnisse
chown -R user01 pictures 		# kompletter Verzeichnisbaum
chown user01 app.conf 			# Benutzeränderung
chown :group01 pictures 		# Gruppenänderung
chgrp group01 pictures 			# Gruppenänderung Alternative
chown user01:admins Pictures 		# Benutzer- und Gruppenänderung

# Spezielle Berechtigungen (suid, sgid, sticky)
-rwsr-xr-x. 1 root root 35504 Jul 16 2010 /usr/bin/passwd 		# setuid
chmod g+s example							# setuid setzen
drwxr-sr-x. 3 root systemd-journal 60 May 18 09:15 /run/log/journal	# setgid
chmod u-s example							# setgid entfernen
drwxrwxrwt. 39 root root 4096 Feb 8 20:52 /tmp				# sticky
chmod +t								# sticky setzen

# Standardberechtigungen umask
umask 0027 # temporärer umask - User RW, Group R, Other -

# initial file permissions	rw-rw-rw-	0666
# umask				-------w-	0002
# result			rw-rw-r--	0664

# initial directory permissions	rwxrwxrwx	0777
# umask				-------w-	0002
# result			rwxrwxr-x	0775

# umask permanent ändern in /etc/profile.d/local-umask.sh
echo "umask 007" >> ~/.bashrc
```

# Kapitel 5: SELinux 
```bash
# Befehle:
getenforce
setenforce
semanage fcontext
restorecon
chcon
getsebool -a
setsebool (-P)
semanage boolean -l
sealert
ausearch -n AVC -ts today

# wichtige Dateien/Ordner:
/etc/selinux/config
/etc/selinux/targeted/contexts/files/ # SELinux DB Kontextverwaltung
/var/log/audit/audit.log
/var/log/messages

# Anzeige von SELinux-Kontexten mit großem Z
ps axZ
ls -Z /var/www

# SELinux- Modus ändern
getenforce 		# Anzeige aktueller Modus
setenforce 0		# Deaktivieren für aktuelle Sitzung
vi /etc/selinux/config 	# Permanentes Ändern

cp -p # Kopieren einer Datei unter Beibehalten des SELinux Kontextes (bei mv wird Kontext beibehalten)

# SELInux-Kontext ändern
chcon -t httpd_sys_content_t /virtual
semanage fcontext -l 						# SELinux-Standarddateikontexte ausgeben
semanage fcontext -l -C						# nur lokale erstellte Kontexte anzeigen
restorecon -vr /virtual 					# zurücksetzen des Kontextes zu Standard (verbosity und rekursiv)
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'	

# Boolsche SELinux-Werte
getsebool -a 				# Anzeige der boolschen Werte inklusive Status
semanage boolean -l			# Aunzeige wie getsebool -a aber zusätzlich Persistenz-Status (Neustart)
getsebool httpd_enable_homedirs		# Anzeige eines Boolschen Wertes
setsebool httpd_enable_homedirs on	# Setzen eines Boolschen Wertes (temporär)
setsebool -P httpd_enable_homedirs 1	# Setzen eines Boolschen Wertes (permanent)

# SELinux Überwachung
yum install setroubleshoot-server 	# notwendiges Installpaket, falls noch nicht vorhanden
tail /var/log/messages			# 1. Fehler wurde in Log eingetragen -> Suche nach sealert und vorgeschlagener Befehl sealert ausführen
sealert -l UUID				# 2. detaillierte Infos zum Fehler anzeigen
ausearch -m AVC -ts recent		# 3. /var/log/audit wird nach AVC Nachrichten mit timestamp recent durchsucht
restorecon -vr /var/www/html		# 4. Standardkontext wiederherstellen
sealert -a /var/log/audit/audit.log	# Alternative Suche im audit log
tail /var/log/audit/audit.log		# Alternative Suche im audit log

```
# Kapitel 6: Tunen der Systemleistung 
```bash

# Befehle:
• ps, kill, killall, pkill, w, pgrep, pstree, jobs
• uptime, lscpu, top
• tuned-adm, (sysctl) 
• nice 

# wichtige Dateien/Ordner:
etc/tuned/tuned-main.conf 	# Konfig des tuned Daemons
/usr/lib/tuned			# Ablage der Tuning Profile (nicht ändern), eigene Profile in /etc/tuned ablegen
/proc/cpu

# Man/help
• man tuned
• man tuned.conf
• man ps
• man nice/renice

# Beenden von Prozessen (Empfehlung: erst SIGTERM-15, dann SIGINT-2, dann SIGKILL-9)
kill -l 		# Auflisten aller verfügbaren Signale
kill PID 		# Kill mit SIGTERM-15
kill -9 PID		# Kill mit SIGKILL-9 | Alternative kill -SIGKILL
pkill sleep		# Beenden mehrere Prozesse auf Basis seines Behehlsnamens | Alternative: killall sleep
pkill -U user01 	# Beenden mehrere Prozesse auf Basis von Kriterien (Terminal, UID, GID, Command, Parent)
jobs -l			# zeigt aktuell laufende Jobs an (Def. Jobs: Prozesse, die über Shell gestartet werden)
kill -SIGSTOP %2	# Stoppt den Job 2 | Alternative kill -19 PID
kill -SIGCONT %2	# Job wird wieder gestartet

# Abmelden von Benutzern/Beenden von Benutzerprozessen
w 						# angemeldete User anzeigen, wie lange aktiv, welche Prozesse
pgrep -l -u user01 -t pts/2 			# Anzeigen aller Prozesse von user01 in Terminal pts/2
ps u $(pgrep sha1sum)				# mit pgrep PID auslesen und über ps anzeigen | Alternative: top
ps -o pid,pcpu,nice,comm $(pgrep sha1sum)	# PID, CPU, Nice, Name von Prozessen | Alternative: top
pkill -u user01 -t pts/2 			# Beenden alle Prozesse von user01 in Terminal pts/2
pstree -p user01				# Prozessbaum von user01 mit PIDs (ggf. Installpaket psmisc notwendig)
pkill -9 -P PID					# Beenden/SIGKILL für alle untergeordneten Prozesse des angegebenen Prozesses

# Überwachen der Prozessaktivität
uptime # load average letzte 1, 5 und 15 min
lscpu # Anzahl der CPUs vom System ermitteln

# From lscpu, the system has four logical CPUs, so divide by 4 (optimale Auslastung ist knapp unter 1)
# load average: 			2.92, 	4.48, 	5.20
# divide by number of logical CPUs:	4 	4 	4
#					---- 	---- 	----
# per-CPU load average: 		0.73 	1.12 	1.30
#					OK	ovLoad	ovLoad

# top Kommando
top	# dynamische Anzeige der Systemprozesse | l, t, m (auf Load, Thread, Memory umschalten)
i 	# nur REssourcenlastige Prozesse anzeigen
u 	# nur Prozesse eines bestimmten Users anzeigen
M|P|N 	# Sortieren nach Mem, CPU, PID
k 	# Prozess killen
r	# renice -> nice Wert ändern

# Tuning-Profile
# ggf. muss tuned Paket installiert werden
# dynamisches Tuning aktivieren in /etc/tuned/tuned-main.conf -> tuned passt tuning Einstellungen dynamisch an
tuned-adm active		# aktives Tuning Profile ausgeben
tuned-adm list			# alle verfügbaren Tuning Profile anzeigen
tuned-adm profile balanced 	# Umstellen auf balanced Profil
tuned-adm profile_info		# Infos zum aktuellen Profil ausgeben
tuned-adm recommend		# empfohlenes Profil ausgeben
tuned-adm off			# Tuning deaktivieren
sysctl vm.dirty_ratio		# tatsächliche Systemeinstellung (vm.dirty_ratio) auslesen

# Prozessplanung
# nice-Wert: je niedriger, desto höher die Prio; Standard ist 0 (in top prio 20); höchste Prio sind -20 (in top prio 0)
top 						# nice Wert und Prio sind je Prozess sichtbar
ps -axo pid,comm,nice,cls --sort=-nice 		# Prozesse mit Sortierung des Nice Wertes
nice -n 15 sleep 60 &				# Prozess mit Nice Wert 15 starten
renice -n -5 PID				# Nice Wert eines Prozesses ändern (unpriv. User können Wert nur erhöhen) 
for i in {1..3}; do sha1sum /dev/zero & done 	# 3 Jobs/Prozesse, die Last generieren, im Hintergrund starten	
```
# Kapitel 7: zukünftige Tasks terminieren 
```bash

# Befehle:
• crontab

# wichtige Dateien/Ordner
/etc/cron.d 			# Ablage cronjob files
/etc/crontab	 		# Beispiel Cronjob
/etc/cron.hourly, ....daily… 	# Ablage ausführbare Scripte , keine crontabs
/etc/anacrontab			# config file für Anacron
/var/spool/anacron	 	# Zeitstempel für Anacron
/etc/systemd/system		# keiner Änderungen, kopieren 
/usr/lib/systemd/system		# ...Ablage und Änderungen hier
/usr/lib/systemd/system/systemd-tmpfiles-clean.timer	# config file tmpfiles-clean

# man/help:
• man 5 crontab, man crontab, man chrond
• man tmpfiles.d (Rangfolge Verzeichnisse, Beispiel)
• man systemd-tmpfiles (Befehle wie --create, --clean)

# Wiederkehrende Benutzerjobs/Crontabs
crontab -l 		# Auflisten aller Jobs für aktuellen User
crontab -r		# Entfernen aller Jobs für aktuellen User
crontab -e		# Bearbeiten aller Jobs für aktuellen User
crontab filename 	# Entfernen und Ersetzen aller Jobs durch Jobs in File

# Wiederkehrende Systemjobs
/etc/crontab 	# Enthält Beispiele, kann auch Cronjobs aufnehmen
/etc/cron.d/	# Empfehlung: Ablage der Cronjobs in Datein in diesem Verzeichnis

#  Formatierung für Cronjobs
Minute (0 - 59) | Stunde (0 - 23) | Tag des Monats (1 - 31) | Monat (1 - 12) | Wochentag (0 - 6; SO bis Sa)
* 			# eine Ausführung in jeder Möglichen Instanz des Feldes
*/7 			# alle 7 Minuten
5,10-13-17		# Job, der 5, 10, 11, 12, 13 und 17 Minuten nach der vollen Stunde ausgeführt wird
Jan, Feb und Mon, Tue	# englische Abkürzungen

# Beispiele für Cronjobs
15 12 11 * Fri command		# Beispiel-Job: jeder 11. des Monats und jeden Freitag um 12:15 ausgeführt
*/5 9-16 * Jul 5 command	# alle 5 min zw. 9-16 Uhr an jedem Freitag im Juli (Start 09:00 Ende 16:55)

# Anacron (Alternative zu Cron, die sicher stellt, dass geplante Jobs immer ausgeführt und nicht übersprungen werden)
# Jobs in den Verzeichnissen /etc/cron.daily, ...weekly, ...monthly werden je nach config in /etc/anacrontab ausgeführt
run-parts 	# Ausführung tagl., wöchntl. und monatl. Jobs in /etc/anacrontab

# Systemd-Timer (Moderne Alternative zu Cron-Jobs)
vim /etc/systemd/system/sysstat-collect.timer 	# timer anlegen und konfiguieren
systemctl daemon-reload				# Daemon neu laden
systemctl enable --now sysstat-collect.timer	# Timer Aktivieren

# Beispiel sysstat-collect.timer (ruft alle 10 min sysstat-collect.service auf:
Activates activity collector every 10 minutes

[Unit]
Description=Run system activity accounting tool every 10 minutes

[Timer]
OnCalendar=*:00/10

[Install]
WantedBy=sysstat.service


# Temporäre Dateien verwalten
systemctl cat systemd-tmpfiles-clean.timer 		# Konfig systemd- tmpfiles-clean.timer der Unit anzeigen
# OnBootSec=15						# Unit wird 15 min nach Boot ausgeführt
# OnUnitActiveSec=1d					# Unit wird nach 1 Tag Betrieb erneut ausgeführt
systemctl daemon-reload 				# Bei Änderungen der Parameter
systemctl enable --now systemd-tmpfiles-clean.timer	# Unit beim Neustart aktivierung und ausführen
/etc/tmpfiles.d/*.conf					# Prio 1: temporäre Speicherorte/Dateien für Benutzer
/run/tmpfiles.d/*.conf					# Prio 2: temporäre Speicherorte/Dateien für Daemons
/usr/lib/tmpfiles.d/*.conf				# Prio 3: temporäre Speicherorte/Dateien für RPM Pakete
systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf	# Temporäre Verzeichnisse/Dateien löschen -> eigene Konfig Datei 
systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf	# Temporäre Verzeichnisse/Dateien anlegen -> eigene Konfig Datei 

```
# Kapitel 8: Installieren von SW Paketen
```bash
# Befehle:
• dnf list, dnf search, dnf info, dnf provides, dnf install, dnf update, dnf remove, dnf group list
• dnf repolist all, dnf config-manager --enable rhel-server-debug-pms, dnf update

# wichtige Dateien/Ordner
/etc/yum.repos.d	# Ablage von .repo -Dateien zum Hinzufügen von Repositorys
/etc/dnf/dnf.conf	# Nicht empfohlen: Ablage von Repos über [repository]-Abschnitt

# man/help
• man dnf, man dnf -config-manager, man dnf.conf

# DNF Befehle (rpm berücksichtigt keine Abhängigkeiten)
dnf list 'http*'		# Anzeige installierte und verfügbare Pakete
dnf search all 'webserver'	# Suche nach Paketen mit bestimmten Schlüsselwort
dnf info PACKAGENAME		# detaill. Infos zum Paket, Speicherplatz (apt show PACKAGENAME)
dnf provides PATHNAME		# Suche nach Paketen, die bestimmten Pfad bereitstellen (z.B. /var/www/html)
dnf install PACKAGENAME	-y	# Installation eines Paketes (silent)
dnf remove PACKAGENAME		# Deinstallation Paket und abhängige Pakete
dnf update PACKAGENAME		# Abruf neuere Version und Installation eines Paketes inklusive Abhängigkeiten
dnf list kernel			# Anzeige installierte und verfügbare Kernel
dnf group list			# Anzeige installierter und verfügbarer Gruppen
dnf group list hidden		# Anzeige installierter und verfügbarer Gruppen (inklusive versteckter Gruppen)
dnf group info GROUPNAME	# detaill. Infos zur Gruppe, Pakete usw.
dnf group install GROUPNAME	# Installation einer Gruppe
tail -5 /var/log/dnf.rpm.log	# Transaktionen Install und Remove (tail -5 /var/log/yum.log)
dnf history
dnf history undo 6		# Kehrt 6. Transaktion um
dnf history info 3		# Infos zur Transaktion

# Repositorys
dnf repolist all					# Auflistung aller verfügbaren Repos
dnf repoinfo epel					# Anzeige Infos zu einem Repo
dnf list --repo=updates					# Anzeiger aller verfügbaren Pakete in dnf
yum list --disablerepo="*" --enablerepo=updates		# Anzeiger aller verfügbaren Pakete in yum
grep ^ /etc/apt/sources.list /etc/apt/sources.list.d/*	# Auflistung aller installierter Repos in Ubuntu
dnf config-manager --enable rhel-9-server-debug-rpms	# Aktivieren/Deaktivieren von Repos (yum-config-manager --enable epel)
dnf config-manager \					# Hinzufügen eines neuen Repos (.repo wird in /etc/yum.repos.d/ erzeugt)
--add-repo="https://dl.fedoraproject.org/pub/epel/9/Everything/x86_64/"

# zusätzliche RPM Befehle
rpm -q PACKAGENAME

```
# Kapitel 9: Basic Storage  
```bash
# Befehle:
• lsblk -fp, parted, udevadm settle, mkfs.xfs, mkswap, mount, free -h, swapon -a
# wichtige Dateien/Ordner
• /etc/fstab

# man/help
• man fstab

# Manuelles Mounten eines FS
mount UUID="efd314d0-b56e-45db-bbb3-3f32ae98f652" /mnt/data	# manuelles Mounten eines FS mit UUID
mount /dev/vda4 /mnt/data					# manuelles Mounten (nicht empfohlen) 
lsblk -fp							# Verfügbare Block Devices anzeigen mit UUID und Mountpath
umount /mnt/data						# unmount
lsof /mnt/data							# Anzeige aller Prozesse, die auf FS zugreifen
			
# Partitionen und FS 
parted /dev/sdb mklabel gpt	# Ein Disk- Label erstellen ->  GPT / Definieren des GPT-Partitionsschema
parted /dev/sdb			# Partition auf dem Device erstellen (interaktiver Modus)
	mkpart
	primary			# Name
	xfs			# FS-Typ
	2048s			# Start
	2GB			# End
parted /dev/vdb mkpart primary xfs 2048s 2GB	# Alternativ zum interaltiven Modus
parted /dev/sdb print				# Partitionen anzeigen, verifizieren
udevadm settle					# neue Partition im System registrieren/ in /dev/* anzeigen
mkfs.xfs /dev/sdb1				# neue Partition mit xfs Filesystem formatieren
mkdir /backup					# neuen Ordner für den Mountpoint erstellen
lsblk -fp /dev/sdb				# UUID des Device herausfinden
vim /etc/fstab					# Eintrag in /etc/fstab vornehmen
mount /dev/vda4 /mnt/data			# Alternative  fstab: temporäres Einbinden
systemctl daemon-reload				# systemd daemon updaten um neuen Eintrag der fstab zu übernehmen
mount /backup					# neues FS mit fstab mounten -> würde einen Fehler werfen, wenn fstab nicht korrekt
mount | grep sdb1				# überprüfen, ob neues FS in /archive gemounted ist
systemctl reboot
parted /dev/vdb rm 1				# Partition löschen

# SWAP-Space 
parted /dev/sdb			# SWAP-Partition ersten (interaktiver Modus)
	mkpart
	swap1			# Name
	linux-swap		# FS-Typ
	1001MB			# Start
	1257MB			# End
parted /dev/sdb mkpart swap1 linux-swap 2000M 2512M
parted /dev/sdb mkpart swap2 linux-swap 2512M 3024M
parted /dev/sdb print		# Partitionen anzeigen, verifizieren
udevadm settle			# neue Partition im System registrieren/ in /dev/* anzeigen
mkswap /dev/sdb2		# neue Partition mit SWAP-space formatieren
mkswap /dev/sdb3
UUID=87976166-4697-47b7-86d1-73a02f0fc803 swap swap pri=10 0 0	# fstab anpassen mit Prio
systemctl daemon-reload		# den systemd daemon updaten um die euen Eintrag der fstab zu übernehmen
free -h				# Speicher und SWAP-Space anzeigen
swapon -a			# SWAP persistent aktivieren (alle Einträge in fstab)
swapon --show	
swapon /dev/sdb2		# Alternative: SWAP temporär aktivieren
swapoff /dev/sdb2		# SWAP deaktivieren 

# fstab 
1. UUID	| 2.Mountpoint | 3. FS-Typ |					# Felder
4. comma-seperated Liste der Optionen |
5. dump (Backup) | 6. fsck-Reihenfolgefeld 				
UUID=7a20315d-ed8b-4e75-a5b6-24ff9e1f9838 /dbdata xfs defaults 0 0	# Beispiel xfs 
UUID=39e2667a-9458-42fe-9665-c5c854605881 swap swap pri=4 0 0 		# Beispiel Swap mit Prio 4
92.168.178.63:/volume1/music /home/sascha/music nfs defaults 0 0 	# Beispiel Synology NAS 
systemctl daemon-reload							# Konfig wird geladen
mount MOUNTPOINT 							# Überprüfung ob fstab Eintrag korrekt
```

# Kapitel 10: Storage Stack  
```bash

# Klassisch: 	Harddrive (sdb) -> Partition/PD (sdb1) 	-> File System -> mounten
# PVM: 		Harddrive (sdb) -> Partition/PD (sdb1)-> PD -> PV -> VGs -> LV -> File System -> mounten

# https://www.youtube.com/watch?v=scMkYQxBtJ4

# Identifizieren der Systemplatten
lsblk -fp
blkid

#PDs vorbereiten: Erstellen eines GPT Labels, 2 Partitionen (xfs), Setzen von lvm flags, Registrieren der Partitionen beim Kernel
parted /dev/sdb mklabel gpt mkpart primary 2048s 1GB
parted /dev/sdb mkpart primary 1GB 2GB
parted /dev/sdb set 1 lvm on
parted /dev/sdb set 2 lvm on
udevadm settle

# LV erstellen
pvcreate /dev/sdb1 /dev/sdb2				# PV erstellen: PD als PV labeln
vgcreate vg01 /dev/sdb1 /dev/sdb2			# VG erstellen vg Name, PV1, PV2...
lvcreate -n lv01 -L 300M vg01				# LV erstellen (in MB)
lvcreate -n lv01 -l 32  vg01				# LV erstellen (in Physical Extends)
lvcreate --type vdo --name vdo-lv01 --size 5G vg01	# (LVM VDO erstellen)
mkfs -t xfs /dev/vg01/lv01				# Filesystem auf LV erstellen 
/dev/vg01/lv01 /mnt/data xfs defaults 0 0		# in fstab einbinden
mount -a
pvdisplay /dev/sdb1					# Anzeigen der erstellten Volumes
vgdisplay vg01 						
lvdisplay /dev/vg01/lv01

# eine VG vergrößern (zuerst neue Partition anlegen)
parted /dev/sdb set 3 lvm on
udevadm settle
pvcreate /dev/vdb3
vgextend vg01 /dev/vdb3

# ein LV vergrößern (um 500 MB bzw. auf 700 MB)
lvextend -L +500M /dev/vg01/lv01
lvextend -L 700M /dev/vg01/lv01

# XFS FS auf die Größe des LV erweitern
xfs_growfs /mnt/data/

# EXT4 FS auf die Größe des LV erweitern
resize2fs /dev/vg01/lv01

# Swap erweitern (lv muss existieren)
swapoff -v /dev/vg01/swap
lvextend -L +300M /dev/vg01/swap
mkswap /dev/vg01/swap
swapon /dev/vg01/swap

# VG Speicher reduzieren
pvmove /dev/sdb1 #Daten von sdb1 werden auf andere PDs verteilt
vgreduce vg01 /dev/sdb1

# LVM Storage entfernen
umount /mnt/data
lvremove /dev/vg01/lv01
vgremove vg01
pvremove /dev/vdb1 /dev/vdb2

# Stratis - stratis create pool /dev/sdb hat nicht funktioniert!
```
# Kapitel 11: Kontrolldienste und Bootvorgang 
```bash

# alle unit types von systemctl anzeigen
systemctl -t help

# Module anzeigen, die geladen und active sind
systemctl
systemctl list-units --type=service	# alle systemctl service units anzeigen (nur active)
systemctl list-units --type=service --all # auch state inactive 
systemctl list-units --type=service --state=inactive # nur state inactive
systemctl list-unit-files # alle installiertem und nicht geladene Units
systemctl status sshd.service	# status einer Unit im Detail anschauen
systemctl is-active sshd.service # true|false
systemctl is-enabled sshd.service
systemctl is-failed sshd.service
systemctl --failed --type=service

# starten, stoppen und restarten von Services (ssh oder auch sshd.service möglich)
systemctl start sshd
systemctl stop sshd.service
systemctl restart sshd.service
systemctl reload sshd.service 			# reload: keine Unterbrechung, PID bleibt, nicht bei jedem Service
systemctl reload-or-restart sshd.service

# Abhängigkeiten von Services
systemctl list-dependencies sshd.service		# welche Services werden benötigt, um Service zu starten
systemctl list-dependencies sshd.service --revers	# welche anderen SRV sind vom SRB abhängig

# Maskieren von Services: kein Boot und kein manuelles starten möglich (härteres Disable)
systemctl mask sendmail.service
systemctl unmask sendmail

# Starten, Stoppen, Booten von Services
systemctl enable sshd.service		# beim Booten starten
systemctl enable --now sshd.service	# zusätzlich gleich starten
systemctl start sshd.service
systemctl disable sshd.service		# beim Booten nicht starten
systemctl disable --now sshd.service	# zusätzlich gleich beenden
systemctl stop sshd.service

# Rechner neu starten, ausschalten
systemctl reboot 	# Alternative: init 6
systemctl poweroff	# Alternative: init 0

# Ziele/Targets
systemctl isolate multi-user.target		# Target zur Laufzeit auswählen
systemctl get-default				# default Profil anzeigen
systemctl set-default graphical.target		# default Profil setzen
systemctl cat graphical.target			# zeigt config-files der Unit an (u.a. ob man es isolieren kann)

# ein anderen Ziel während des Bootvorgangs: 
1. bootloader mit beliebiger Taste (außer Enter) unterbrechen
2. Standard Kernel auswählen und e drücken
3. in Zeile Linux am Ende folgendes anfügen: systemd.unit=rescue.target
4. STRG + X Drücken

# Root Passwort zurücksetzen
1. beim Bootvorgang (Kernel-Auswahl) beliebige Taste drücken 
2. Rescue-Kernel auswählen und e drücken
3. rd.break in linux Zeile anhängen
4. STRG + X Drücken, dann Enter
5. Stellen Sie /sysroot erneut mit Lese-/Schreibberechtigungen bereit
sh-5.1# mount -o remount,rw /sysroot
6. Wechseln Sie in ein chroot-Jail, in dem /sysroot als Root der Dateisystemstruktur behandelt wird.
sh-5.1# chroot /sysroot
7. neues PW vergeben
sh-5.1# passwd root
8.Stellen Sie sicher, dass alle nicht gekennzeichneten Dateien, einschließlich /etc/shadow an
diesem Punkt, während des Boot-Vorgangs wieder gekennzeichnet werden.
sh-5.1# touch /.autorelabel
9. Beenden
exit exit

# Überprüfen von Logs
# boot-log-Ablage default ist /run/log/journal
# für persistente Ablage in /var/log/journal muss Storage=persistent in /etc/systemd/journald.conf gesetzt werden
journalctl -b -1 -p err		# Fehlerlogs des vorherigen Bootvorgangs

### Boot Probleme beheben - Early Debug Shell
# beim Bootvorgang STRG + Alt + F9 drücken
# Boot vorgang bei Kernelauswahl unterbrechen und an Zeile mit Linux systemd.debug-shell anhängen

### Emergency und Rescue Targets
# emergency: root FS bleibt read-only
# rescue: wartet auf sysinit.target -> initialisiert mehr vom System wie logging und file systems
systemd.unit=emergency.target
systemd.unit=rescue.target

# hängengebliebene Jobs identifizieren
systemctl list-jobs

### Fehlerhaft fstab korrigieren
# normaler Weise startet System im Emergency Modus, ich schaue mir an, welche FS gemountet sind und ob /root mit rw gemountet
mount
# wenn root FS nicht mit rw gemountet ist, kann ich fstab nicht anpassen
mount -o remount,rw /
# ich versuche, alle FS in fstab zu mounten -> so kann ich Fehler identifizieren
mount --all
mount: /mnt/mountfolder: mount point does not exist.
# anschließend korrigiere ich den Fehler und registriere neue fstab
systemctl daemon-reload
mount --all
```
# Kapitel 12: Analysieren und Speichern von Protokollen
```bash
# Befehle:
logger, journalctl, timedatectl

# wichtige Dateien/Ordner
/etc/rsyslog.conf	# config von Logging Regeln
/etc/chrony.conf	# config Zeitsync, NTP
/var/log		# persistente Speicherung von logs

# man/help
• man rsyslog.conf, journalctl, systemd.journal-fields, systemd.time

# Syslog:
# Regeln für rsyslog unter /etc/rsyslog.conf bzw. /etc/rsyslog.d (*.conf) --> wie werden logs behandelt?
# Pri aufsteigend: debug, info, notice, warning, err, crit, alert, emerg

logger -p local7.notice "Log entry created on host"	# eine Nachricht an rsyslog schicken -> boot.log

# spezifische Log-Nachrichten an neue Logfile senden
echo "*.debug /var/log/messages-debug" > /etc/syslog.d/debug.conf	# neue config-Datei erstellen:
systemctl restart rsyslog						# rsyslog neu starten
logger -p user.debug "Debug Message Test"				# Testnachricht schicken
logger -p user.info "Info Message Test"				# Loggt alles ab level debug und höher

# Systemjournal
journalctl -n 5 -p err		# letzten 5 Nachrichten (nur err Prio) im journalctl betrachten
journalctl -f			# journalctl log fortführend
journalctl -u sshd.service 	# nur Nachrichten einer speziellen Unit auflisten
journalctl --since today	# verschiedene Zeitbereiche
journalctl --since "2022-03-11 20:30" --until "2022-03-14 10:00"
journalctl --since "-1 hour"
journalctl -o verbose		# zusätzliche Details einblenden

# gebräuchliche Felder zum Suchen in journalctl (man systemd.journal-fields):
journalctl _COMM=dbus-daemon			# command name
journalctl _PID=1				# Prozess PID
journalctl _UID=81				# UID of the user that runs the process.
journalctl _SYSTEMD_UNIT=sshd.service _PID=2110	# systemd unit that started the process
# _EXE is the path to the executable file for the process.

# System Journal wird beim Reboot gelöscht /run/log -> zum Persistieren /etc/systemd/journald.conf anpassen
# storage=persistent (logs werden in /var/log/journal gespeichert)
# storage=volatile (Logs werden (nicht persistent) in /run/log/journal gespeichert
# storage=auto (wenn /var/log/journal existiert, wird darin gespeichert, sonst /run/log/journal)
# storage=none (es werden keine Logs gespeichert)

# nach Änderung journald neustarten
systemctl restart systemd-journald

# -nur nach Persistierung- Anzeige aller Logs des 2. Systemboots
journalctl -b 2
journalctl --list-boots

# Verwalten lokaler Uhrzeiten und Zeitzonen
timedatectl 					# Überblick aktuelle Zeiteinstellungen (Zeit, Zeitzone, NTP)
timedatectl list-timezones			# verfügbare Zeitzonen anzeigen
tzselect					# Dialog korrekten Zeitzone starten (wird nicht in config übernommen)
timedatectl set-timezone America/Phoenix	# Ändern der aktuellen Zeitzone
timedatectl set-timezone UTC			# Ändern auf UTC
timedatectl set-ntp false			# NTP deaktivieren

# Konfiguration von chronyd in /etc/chrony.conf
# default ist ein Pool an ntp Servern, habe ich einen eigenen NTP Server im Netzwerk, muss ich "server..." angeben
chronyc sources -v				# NTP Quellen anzeigen:

```
############################## Kapitel 13: Networking ##########################################################
```bash
# Befehle:
ip, ping, tracepath, traceroute, ss

# wichtige Dateien/Ordner
/etc/ports	# Standardports

# man/help
ip-link(8), ip-address(8), ip-route(8), ip(8), ping(8),tracepath(8), traceroute(8), ss(8) und netstat(8)
# weiter auf S. 482 /home/sascha/Dropbox/Dropbox_Sync/IT-Fortbildung/Linux_Admin

ip link show		# Auflisten aller Netzwerkschnittstellen
ip addr show ens3	# IP-Adressen eines Gerätes anzeigen
ip -br addr list 	# Auflisten aller IP Adressen eines Servers
ip -s link show ens3	# Performance des Netzwerks anzeigen
ping6 2001:db8:0:1::1	# Verbindung zu anderen Hosts testen (ipv6)
ping -c3 192.0.2.254	# Verbindung zu anderen Hosts testen (ipv4) - nur 3 Pakete
ping6 ff02::1%ens4	# ping an multicast Gruppen (ens4 oder je nach Name des Gerätes)
ip route		# Routing anzeigen
ip -6 route

# noch zu Bearbeiten:
# was ist eine link-local address?
# Was ist link-local all-nodes multicast group?
# Was ist Global Scope?

# Nachverfolgen von Routen
tracepath access.redhat.com
traceroute -I google.com	# muss ggf. installiert werden; UPD Standard, -I ICMP, -T TCP
tracepath6 2001:db8:0:2::451	# IPv6

# Aufruf von Socket-Statistiken:
ss -tulpna 		# ersetzt netstat 
netstat -tulpna		# nicht immer installiert
# -n Zahlen anstatt von Namen für Ports
# -t -u TCP, UPD
# -l nur abhörende Sockets
# -a alle (abhörend und etablierte) Sockets
# -p Prozess, der Sockets verwendet

# Check, welcher Prozess einen Port verwendet
sudo lsof -i :8080

# Portweiterleitung des lokalen Ports 8080 auf den entfernten Server 128.140.77.235  
# dort wird die Anfrage an die IP 172.19.0.2 auf Port 80 weitergeleitet.
# Voraussetzung: der eigene public key muss auf dem remote rechner unter /home/cnbc/.ssh/autorized_keys abgelegt sein
ssh -fN -L 8080:172.19.0.2:80 cnbc@128.140.77.235


# Netzwerk Config Files sind in /etc/NetworkManager/system-connections/

# Status aller NW-Geräte anzeigen
nmcli dev status

# Liste aller NW-Verbindungen anzeigen (Abkürzungen wie con möglich) -- hier nur aktive 
nmcli con show --active 


```
# Kapitel 14: Network-Attached Storage 
```bash
```
# Kapitel 15: Network Security 
```bash
```
# Kapitel 16: Container 
```bash
```


# Zusätzliche Commandos
```bash
date -d "+45 days" -u # Datum 45 Tage in Zukunft in UTC anzeigen

# Suche nach Dateien und Verzeichnissen
find . -type f -name '*.log.gz' -size -1M		# kleiner als 1 MB im aktuellen Ordner
find /var/log -type f -name '*.log' -size +100c		# größer als 100 Byte unterhalb /var/log
find . -type d						# Suche nach Verzeichnis
find music/ -type d -exec chmod 755 {} \;		# Kombi aus chmod und find

# grep
grep -c # zählt Vorkommen einer bestimmter Zeichenkette
grep -v '^system' 			# Alle Vorkommen, die nicht mit system beginnen
# sed - command
sed 's/alt/neu/' file 			# Änderung nur in STOUT; s-stringsearch;
sed 's!path://alt!path://neu!' file 	# Änderung Trenner
sed 's/delete//'			# Löschen von Strings
sed -ri -e "/^Listen 80/c\Listen ${PORT}" /etc/httpd/conf/httpd.conf # Suchen und Ersetzen von Listen 80
- r (Extended Regular Expression Mode) Verwednung von Klammern ohne Escapezeichen
- i in place: direktes Bearbeiten ohne Ausgabe in STOUT
- e Befele zu eingabe
^ "/^Listen 80/ - Markiert den Anfang einer Zeile

# jq
# tar
tar -xvf /colors.tar -C /folder 		# entpackt ein Archiv in einen (vorhandenen) Ordner
tar -cvf new.tar sourcefolder 			# erstellt neues Archiv von der Quelle

# TCPDump (Netzwerkdiagnosewerkzeug)
sudo tcpdump -i eth0 -A -n port 80 		# Überwachung des Geräts (-i eth0) auf Port 80
						# -n disables DNS
						# -A Ausgabe Header und Daten in ASCII (statt Hex)

# curl
curl google.com					# Get Anfrage (identisch mit curl -X GET google.com)	
curl -I -v google.com				# I - nur Header keinen Inhalt | v - verbosity
curl -X POST google.com				# POST Anfrage
curl -sL					# s-silent (keine Meldungen), L- Redirect erlaubt

# tcpdump
# Bridget hat die URLS damit bekommen, auf die ein Alpine Image beim Upgrade Befehl zugreift

# nslookup
# wget
```
