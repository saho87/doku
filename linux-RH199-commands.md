Formatierung:
https://docs.github.com/de/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax


# Kapitel 1: System Access und Support 

```bash 
# Befehle:
ssh-keygen, ssh-copy-id, ssh-add, ssh

# wichtige Dateien/Ordner:
/home/user/.ssh

ssh-keygen -f .ssh/key-with-pass
ssh-copy-id -i .ssh/key-with-pass.pub user@remotehost	# Ablage in /.ssh/authorized_keys
ssh-keygen -f .ssh/key-with-pass

# SSH-Agent wird gestartet und  ENV, die vom Agenten gesetzt werden, in aktuellen Shell-Umgebung aktiviert
eval $(ssh-agent)
# Hinzufügen eines priv Keys (passphrase muss dann nicht mehr eingegeben werden)		
ssh-add .ssh/key-with-pass				

# SSH-Verbindung mit Debugging und speziellem Key
ssh -v -i .ssh/key-with-pass user@remotehost	
```
# Kapitel 2: Verwalten von Dateien über Befehlszeile
```bash
stat				# Zeitstempel anzeigen
```
# Kapitel 3: Verwalten lokaler Benutzer und Gruppen
```bash
# Befehle:
id, su -, sudo -i, sudo 
useradd, userdmod, userdel - r, passwd
groupadd, groupmod, groupdel, newgrp
chage, date

wichtige Dateien/Ordner:
/etc/passwd
/etc/group
/etc/login.defs
/etc/sudoers.d/ und /etc/sudoers

```
# Kapitel 4: Steuern des Zugriffs auf Dateien
```bash
# Befehle:
chmod, chown, (chgrp)
umask

# wichtige Dateien/Ordner:
/etc/profile, /etc/profile.d, /etc/bashrc (zusätzlich im Home)

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
• /etc/selinux/config
• /var/log/audit/audit.log
• /var/log/messages

```
# Kapitel 6: Tunen der Systemleistung 
```bash
• Command Substitution (https://www.youtube.com/watch?v=hMGHqDz6fPc)
ps -o pid,pcpu,nice,comm \
$(pgrep sha1sum;pgrep md5sum)

# Befehle:
• ps, kill, killall, pkill, w, pgrep, pstree, jobs
• uptime, lscpu, top
• tuned-adm, (sysctl) 
• ps axo pid, comm, nice, cls --sort=-nice
• ps -o pid,pcpu,nice,comm $(pgrep sha1sum;pgrep md5sum)
• ps u $(pgrep sha1sum)
• nice sleep 600 &, renice -n 12 {pid}

# wichtige Dateien/Ordner:
• etc/tuned/tuned-main.conf
• /usr/lib/tuned
• /proc/cpu

# Man/help
• man tuned
• man tuned.conf
• man ps
• man nice/renice
```
# Kapitel 7: zukünftige Tasks terminieren 
```bash
# Befehle:
• crontab

# wichtige Dateien/Ordner
• /etc/cron.d -> da cronjob files rein
• /etc/crontab
• /etc/cron.hourly, /etc/cron.daily… -> ausführbare Scripte werden automatisch ausgeführt, keine crontabs
• /etc/anacrontab, /var/spool/anacron
• /etc/systemd/system, (/usr/lib/systemd/system)
• /usr/lib/systemd/system/systemd-tmpfiles-clean.timer

# man/help:
• man 5 crontab, man crontab, man chrond
• man tmpfiles.d (Rangfolge Verzeichnisse, Beispiel)
• man systemd-tmpfiles (Befehle wie --create, --clean)

```
# Kapitel 8: Installieren von SW Paketen
```bash
# Befehle:
• dnf list, dnf search, dnf info, dnf provides, dnf install, dnf update, dnf remove, dnf group list
• dnf repolist all, dnf config-manager --enable rhel-server-debug-pms, dnf update

# wichtige Dateien/Ordner
• /etc/yum.repos.d

# man/help
• man dnf, man dnf -config-manager, man dnf.conf

```
# Kapitel 9: Basic Storage  
```bash
# Befehle:
• lsblk -fp, parted, udevadm settle, mkfs.xfs, mkswap, mount, free -h, swapon -a
# wichtige Dateien/Ordner
• /etc/fstab

# man/help
• man fstab
### Partitionen und FS 

	# Ein Disk- Label erstellen ->  GPT / Definieren des GPT-Partitionsschema
	parted /dev/sdb mklabel 

	# Partition auf dem Device erstellen
	parted /dev/sdb
		mkpart
		primary
		xfs
		2048s
		2GB

	# Partitionen anzeigen, verifizieren
	parted /dev/sdb print

	# neue Partition im System registrieren/ in /dev/* anzeigen
	udevadm settle

	# neue Partition mit xfs Filesystem formatieren
	mkfs.xfs /dev/sdb1

	# neuen Ordner für den Mountpoint erstellen
	mkdir /backup

	# UUID des Device herausfinden
	lsblk -fp /dev/sdb

	# Eintrag in /etc/fstab vornehmen
	vim /etc/fstab

	# den systemd daemon updaten um die config/neuen Eintrag der fstab zu übernehmen
	systemctl daemon-reload

	# neues FS mit fstab mounten -> würde einen Fehler werfen, wenn Eintrag in /etc/fstab nicht korrekt
	mount /backup

	# überprüfen, ob neues FS in /archive gemounted ist
	mount | grep sdb1

	#reboot
	systemctl reboot


### SWAP-Space 


	# SWAP-Partition erstellen
	parted /dev/sdb
		mkpart
		swap1
		linux-swap
		1001MB
		1257MB
	parted /dev/sdb mkpart swap1 linux-swap 2000M 2512M
	parted /dev/sdb mkpart swap2 linux-swap 2512M 3024M

	# Partitionen anzeigen, verifizieren
	parted /dev/sdb print

	# neue Partition im System registrieren/ in /dev/* anzeigen
	udevadm settle

	# neue Partition mit SWAP-space formatieren
	mkswap /dev/sdb2
	mkswap /dev/sdb3

	# fstab anpassen mit Prio
	UUID=87976166-4697-47b7-86d1-73a02f0fc803 swap swap pri=10 0 0

	# den systemd daemon updaten um die config/neuen Eintrag der fstab zu übernehmen
	systemctl daemon-reload

	# Speicher und SWAP-Space anzeigen
	free -h

	# SWAP persistent aktivieren (alles im fstab)
	swapon -a
	swapon --show


	# Alternative: SWAP temporär aktivieren
	swapon /dev/sdb2

	# SWAP deaktivieren 
	swapoff /dev/sdb2



### fstab 

# Infos: man fstab

UUID=7a20315d-ed8b-4e75-a5b6-24ff9e1f9838 /dbdata xfs defaults 0 0

1. UUID	
2.Mountpoint 
3. FS-Typ 
4. comma-seperated Liste der Optionen
	default liefert set an allgemein genutzten Optionen

UUID=39e2667a-9458-42fe-9665-c5c854605881 swap swap defaults 0 0

# Einbinden des Synology NAS
92.168.178.63:/volume1/music /home/sascha/music nfs defaults 0 0


1. UUID 2. swap (eigentlich MP-hier Platzhalter) 3.

```
# Kapitel 10: Storage Stack  
```bash

# Klassisch: 	Harddrive (sdb) -> Partition/PD (sdb1) 						-> File System -> mounten
# PVM: 			Harddrive (sdb) -> Partition/PD (sdb1)-> PD -> PV -> VGs -> LV -> File System -> mounten

# https://www.youtube.com/watch?v=scMkYQxBtJ4

# Identifizieren der Systemplatten
lsblk
blkid

#PDs vorbereiten: Erstellen eines GPT Labels, 2 Partitionen (xfs), Setzen von lvm flags, Registrieren der Partitionen beim Kernel
[root@host ~]# parted /dev/sdb mklabel gpt mkpart primary 2048s 1GB
[root@host ~]# parted /dev/sdb mkpart primary 1GB 2GB
[root@host ~]# parted /dev/sdb set 1 lvm on
[root@host ~]# parted /dev/sdb set 2 lvm on
[root@host ~]# udevadm settle

# PV erstellen: PD als PV labeln
[root@host ~]# pvcreate /dev/sdb1 /dev/sdb2

# VG erstellen
# erstes Argument ist der vg Name, gefolgt von einem oder mehreren PV
[root@host ~]# vgcreate vg01 /dev/sdb1 /dev/sdb2

# LV erstellen (selbe Größe einmal in MB, einmal in Physical Extends PE)
[root@host ~]# lvcreate -n lv01 -L 300M vg01
[root@host ~]# lvcreate -n lv01 -l 32  vg01

# (LVM VDO erstellen)
[root@host ~]# lvcreate --type vdo --name vdo-lv01 --size 5G vg01

# Filesystem auf LV erstellen 
[root@host ~]# mkfs -t xfs /dev/vg01/lv01

# in fstab einbinden
/dev/vg01/lv01 /mnt/data xfs defaults 0 0
mount -a

# LVM Status (alle Geräte anzeigen ohne Argument)
[root@host ~]# pvdisplay /dev/sdb1
[root@host ~]# vgdisplay vg01 
[root@host ~]# lvdisplay /dev/vg01/lv01

# eine VG vergrößern (zuerst neue Partition anlegen)
[root@host ~]# parted /dev/sdb set 3 lvm on
[root@host ~]# udevadm settle
[root@host ~]# pvcreate /dev/vdb3
[root@host ~]# vgextend vg01 /dev/vdb3

# ein LV vergrößern (um 500 MB bzw. auf 700 MB)
[root@host ~]# lvextend -L +500M /dev/vg01/lv01
[root@host ~]# lvextend -L 700M /dev/vg01/lv01

# XFS FS auf die Größe des LV erweitern
[root@host ~]# xfs_growfs /mnt/data/

# EXT4 FS auf die Größe des LV erweitern
[root@host ~]# resize2fs /dev/vg01/lv01

# Swap erweitern (lv muss existieren)
[root@host ~]# swapoff -v /dev/vg01/swap
[root@host ~]#lvextend -L +300M /dev/vg01/swap
[root@host ~]# mkswap /dev/vg01/swap
[root@host ~]# swapon /dev/vg01/swap

# VG Speicher reduzieren
[root@host ~]# pvmove /dev/sdb1 #Daten von sdb1 werden auf andere PDs verteilt
[root@host ~]# vgreduce vg01 /dev/sdb1

# LVM Storage entfernen
[root@host ~]# umount /mnt/data
[root@host ~]# lvremove /dev/vg01/lv01
[root@host ~]# vgremove vg01
[root@host ~]# pvremove /dev/vdb1 /dev/vdb2

# Stratis - stratis create pool /dev/sdb hat nicht funktioniert!

# Kapitel 11: Kontrolldienste und Bootvorgang 

# alle unit types von systemctl anzeigen
[root@host ~]# systemctl -t help

# Module anzeigen, die geladen und active sind
[root@host ~]# systemctl

# alle systemctl service units anzeigen (nur active) - die im Memory geladen sind
[root@host ~]# systemctl list-units --type=service

# alle systemctl service units anzeigen (auch state inactive bzw. nur inactive)
[root@host ~]# systemctl list-units --type=service --all
[root@host ~]# systemctl list-units --type=service --state=inactive

# alle systemd units anzeigen (auch die installierten und nicht in Memory geladenen)
[root@host ~]# systemctl list-unit-files

# status einer Unit im Detail anschauen
[root@host ~]# systemctl status sshd.service

# verschiedene Status verifizieren
[root@host ~]# systemctl is-active sshd.service
[root@host ~]# systemctl is-enabled sshd.service
[root@host ~]# systemctl is-failed sshd.service
[root@host ~]# systemctl --failed --type=service

# starten, stoppen und restarten von Services (ssh oder auch sshd.service möglich)
[root@host ~]# systemctl start sshd
[root@host ~]# systemctl stop sshd.service
[root@host ~]# systemctl restart sshd.service

# reload eines Services (gleiche PID, kein Unterbrechen, aber nicht bei allen Services möglich)
[root@host ~]# systemctl reload sshd.service
[root@host ~]# systemctl reload-or-restart sshd.service

# Abhängigkeiten- welche Services werden benötigt, um Service zu starten
[root@host ~]# systemctl list-dependencies sshd.service

# Abhängigkeiten- welche Services sind von Service abhängig
[root@host ~]# systemctl list-dependencies sshd.service --revers

# Maskieren von Services
# Maskieren ist härteres Disable- nach disable, wird dienst nicht beim Booten neugestarten, kann aber manuell gestartet werden
# beim mask kann ich ihn auch nicht manuell starten
[root@host ~]# systemctl mask sendmail.service
[root@host ~]# systemctl unmask sendmail

# 1. Services beim Booten starten bzw. 2. zusätzlich sofort starten oder 3. nur starten
[root@root ~]# systemctl enable sshd.service
[root@root ~]# systemctl enable --now sshd.service
[root@root ~]# systemctl start sshd.service

# 1. Services beim Booten nicht starten bzw. 2. zusätzlich sofort stoppen oder 3. nur stoppen
[root@root ~]# systemctl disable sshd.service
[root@root ~]# systemctl disable --now sshd.service
[root@root ~]# systemctl stop sshd.service

# Rechner neu starten, ausschalten
[root@host ~]# systemctl reboot, poweroff

# ein Ziel zur Laufzeit auswählen/isolieren
[root@host ~]# systemctl isolate multi-user.target

# default Standardziel anzeigen und ändernn
[root@host ~]# systemctl get-default
[root@host ~]# systemctl set-default graphical.target

# zeigt config-files der Unit an (u.a. ob man es isolieren kann)
[user@host ~]$ systemctl cat graphical.target

### ein anderen Ziel während des Bootvorgangs: 
# bootloader mit beliebiger Taste (außer Enter) unterbrechen
# Kernel auswählen und e drücken
# in Zeile Linux am Ende folgendes anfügen: systemd.unit=rescue.target
# STRG + X Drücken

### Root Passwort zurücksetzen
# beim Bootvorgang (Kernel-Auswahl) beliebige Taste drücken 
# Rescue-Kernel auswählen und e drücken
# rd.break in linux Zeile anhängen
# STRG + X Drücken, dann Enter
# Stellen Sie /sysroot erneut mit Lese-/Schreibberechtigungen bereit
sh-5.1# mount -o remount,rw /sysroot
# Wechseln Sie in ein chroot-Jail, in dem /sysroot als Root der Dateisystemstruktur behandelt wird.
sh-5.1# chroot /sysroot
# neues PW vergeben
sh-5.1# passwd root
#Stellen Sie sicher, dass alle nicht gekennzeichneten Dateien, einschließlich /etc/shadow an
#diesem Punkt, während des Boot-Vorgangs wieder gekennzeichnet werden.
sh-5.1# touch /.autorelabel
exit exit

### Überprüfen von Logs
# um persistent boot-logs zu speichern (default ist /run/log/journal) in /var/log/journal muss
# Storage=persistent in /etc/systemd/journald.conf gesetzt werden
# boot logs des vorherigen Boots können (-1) mit folgendem Befehl inspiziert werden:
[root@host ~]# journalctl -b -1 -p err

### Boot Probleme beheben - Early Debug Shell
# beim Bootvorgang STRG + Alt + F9 drücken
# Boot vorgang bei Kernelauswahl unterbrechen und systemd.debug-shell anhängen

### Emergency und Rescue Targets
# emergency: root FS bleibt read-only
# rescue: wartet auf sysinit.target -> initialisiert mehr vom System wie logging und file systems
systemd.unit=emergency.target
systemd.unit=rescue.target

# hängengebliebene Jobs identifizieren
systemctl list-jobs

### Fehlerhaft fstab korrigieren
# normaler Weise startet System im Emergency Modus, ich schaue mir an, welche FS gemountet sind und ob /root mit rw gemountet
[root@host ~]# mount
# wenn root FS nicht mit rw gemountet ist, kann ich fstab nicht anpassen
[root@host ~]# mount -o remount,rw /
# ich versuche, alle FS in fstab zu mounten -> so kann ich Fehler identifizieren
[root@host ~]# mount --all
mount: /mnt/mountfolder: mount point does not exist.
# anschließend korrigiere ich den Fehler und registriere neue fstab
[root@host ~]# systemctl daemon-reload
[root@host ~]# mount --all
```
############################## Kapitel 12: Kontrolldienste und Bootvorgang #####################################
```bash

# Regeln für rsyslog unter /etc/rsyslog.conf bzw. /etc/rsyslog.d (*.conf) --> wie werden logs behandelt?
# man rsyslog.conf
# man journalctl

# eine Nachricht an rsyslog schicken
[root@host ~]# logger -p local7.notice "Log entry created on host"

### spezifische Log-Nachrichten an neue Logfile senden
# neue config-Datei erstellen:
[root@servera ~]# echo "*.debug /var/log/messages-debug" > /etc/syslog.d/debug.conf
# rsyslog neu starten
[root@servera ~]# systemctl restart rsyslog

# letzten 5 Nachrichten (nur mit err Priorität) im journalctl betrachten
[root@host ~]# journalctl -n 5 -p err

# journalctl log fortführend
[root@host ~]# journalctl -f

# nur Nachrichten einer speziellen Unit auflisten
[root@host ~]# journalctl -u sshd.service

# zeitliche Begrenzung der journalctl Messages (format "YYYY-MM-DD hh:mm:ss")
[root@host ~]# journalctl --since today
[root@host ~]# journalctl --since "2022-03-11 20:30" --until "2022-03-14 10:00"
[root@host ~]# journalctl --since "-1 hour"

# zusätzliche Details einblenden
[root@host ~]# journalctl -o verbose

# gebräuchliche Felder zum Suchen in journalctl (nicht in man oder help gefunden):
#• _COMM is the command name.
#• _EXE is the path to the executable file for the process.
#• _PID is the PID of the process.
#• _UID is the UID of the user that runs the process.
#• _SYSTEMD_UNIT is the systemd unit that started the process.
[root@host ~]# journalctl _SYSTEMD_UNIT=sshd.service _PID=2110

# System Journal wird beim Reboot gelöscht /run/log -> zum Persistieren /etc/systemd/journald.conf anpassen
# storage=persistent (logs werden in /var/log/journal gespeichert)
# storage=volatile (Logs werden nicht persistent in /run/log/journal gespeichert
# storage=auto (wenn /var/log/journal existiert, wird darin gespeichert, sonst /run/log/journal)
# storage=none (es werden keine Logs gespeichert)

# nach Änderung journald neustarten
[root@host ~]# systemctl restart systemd-journald

# -nur nach Persistierung- Anzeige aller Logs des 2. Systemboots
[root@host ~]# journalctl -b 2
[root@host ~]# journalctl --list-boots

# Überblick über aktuelle Zeiteinstellungen (aktuelle Zeit, Zeitzone, NTP Einstellungen)
[user@host ~]$ timedatectl

# verfügbare Zeitzonen anzeigen
[user@host ~]$ timedatectl list-timezones

# Dialog zur Identifikation der korrekten Zeitzone starten (wird nicht in config übernommen)
[user@host ~]$ tzselect

# Ändern der aktuellen Zeitzone bzw auf UTC
[root@host ~]# timedatectl set-timezone America/Phoenix
[root@host ~]# timedatectl set-timezone UTC

# NTP deaktivieren
[root@host ~]# timedatectl set-ntp false

# Konfiguration von chronyd in /etc/chrony.conf
# default ist ein Pool an ntp Servern, habe ich einen eigenen NTP Server im Netzwerk, muss ich "server..." angeben

# NTP Quellen anzeigen:
[root@host ~]# chronyc sources -v
```
############################## Kapitel 13: Networking ##########################################################
```bash

# Auflisten aller Netzwerkschnittstellen
[user@host ~]$ ip link show

# IP-Adressen eines Gerätes anzeigen
[user@host ~]$ ip addr show ens3

# Performance des Netzwerks anzeigen
[user@host ~]$ ip -s link show ens3

# Verbindung zu anderen Hosts testen (ipv6 und ipv4)
[user@host ~]$ ping6 2001:db8:0:1::1
[user@host ~]$ ping -c3 192.0.2.254

# ping an multicast Gruppen (ens4 oder je nach Name des Gerätes)
[user@host ~]$ ping6 ff02::1%ens4

# Routing anzeigen
[user@host ~]$ ip route
[user@host ~]$ ip -6 route

# noch zu Bearbeiten:
# was ist eine link-local address?
# Was ist link-local all-nodes multicast group?
# Was ist Global Scope?

# Nachverfolgen von Routen

[user@host ~]$ tracepath access.redhat.com
[root@localhost ~]# traceroute -I google.com
[user@host ~]$ tracepath6 2001:db8:0:2::451

# Aufruf von Socket-Statistiken:
[user@host ~]$ ss -tulpna
[user@host ~]$ netstat -tulpna

# Check, welcher Prozess einen Port verwendet
[user@host ~]$ sudo lsof -i :8080

# Portweiterleitung des lokalen Ports 8080 auf den entfernten Server 128.140.77.235  
# dort wird die Anfrage an die IP 172.19.0.2 auf Port 80 weitergeleitet.
# Voraussetzung: der eigene public key muss auf dem remote rechner unter /home/cnbc/.ssh/autorized_keys abgelegt sein
[user@host ~]$ ssh -fN -L 8080:172.19.0.2:80 cnbc@128.140.77.235


# Netzwerk Config Files sind in /etc/NetworkManager/system-connections/

# Status aller NW-Geräte anzeigen
[user@host ~]$ nmcli dev status

# Liste aller NW-Verbindungen anzeigen (Abkürzungen wie con möglich) -- hier nur aktive 
[user@host ~]$ nmcli con show --active 


```
############################## Kapitel 14: Network-Attached Storage ############################################
```bash
```
############################## Kapitel 15: Network Security ####################################################
```bash
```
############################## Kapitel 16: Container ###########################################################
```bash
```
