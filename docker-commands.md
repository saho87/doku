# Erstellen von containerisierten Services
```bash


# Starten eines Containers -> Alternativ: run = create + start -a
# it -Interaktiv, d – im Hintergrund, rm wird nach Ausführung gelöscht, name – Vergabe eines Namens, p- Portweiterleitung
docker run -d --rm --name my_container -p 8080:80 httpd:latest       # Portmapping 8080 des Hosts -> 80 des Containers
docker run -e GREET=Hello -e NAME=RedHat alpine  printenv GREET NAME # Nutzung von Umgebungsvariablen
-e MYSQL_PASSWORD=mypa55 -e MYSQL_DATABASE=items \
-e MYSQL_ROOT_PASSWORD=r00tpa55  \ -d registry.redhat.io/rhel8/mysql-80:1
docker run registry.redhat.io/rhel8/httpd-24 ls /tmp                 # httpd wird nicht gestartet (dafür wird ls /tmp ausgeführt)
docker restart my-httpd                                              # gestoppter Container wird neu gestartet

# Beenden von Containern
docker ps (-a)                                # aller laufenden (a- gelaufenen) Container ausgeben
docker stop <CID>                             # Stoppen (kann wieder gestartet werden)
docker kill (-s SIGKILL) <CID>                # Stoppen unter Nutzung von Unix-Signalen
docker system prune                           # Löscht alle gestoppte Container
docker rm <CID>                               # Löscht einen gestoppten Container

# Informationen zu Containern
docker logs <CID>            # Listet alle Logs zu einem Container auf
docker inspect <CID>         # detaillierte Infos zu Container

# Prozess in bestehenden Container starten
docker exec -it <CID> sh                 # Führt eine 2. Prozess in einem laufenden Container aus (hier öffnen einer shell)
docker exec -it <CID> /bin/bash -c 'ls'  # FÜhrt im Container ein Bash Commando aus
docker run -it <CID> /bin/bash           # Erstellt und führt neuen Container aus und gibt uns die Shell zum Zugriff
docker cp ./file 60ed77c57bb5:.          # Kopieren einer Datei vom Host in den Container


# Einbinden von Volumes
mkdir -pv /home/student/local/mysql                        # Verzeichnis erstellen
sudo semanage fcontext -a \                                # SELinux-Kontext erstellen
 -t container_file_t '/home/student/local/mysql(/.*)?'
sudo restorecon -R /home/student/local/mysql               # Kontext anwenden
ls -ldZ /home/student/local/mysql                          # Kontext verifizieren
podman unshare chown 27:27 /home/student/local/mysql       # Eigentümer des Verzeichnisses auf mysql-Container-User ändern
docker run --name persist-db \                             # Container Start
-d -v /home/student/local/mysql:/var/lib/mysql/data \
-e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 \
-e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 \
registry.redhat.io/rhel8/mysql-80:1

# Zuordnung von Ports
docker run -d --name apache1 -p 8080:8080 httpd              # Host 8080 -> Container 8080
docker run -d --name apache2 -p 127.0.0.1:8081:8080 httpd    # Binden einer IP (hier localhost)
docker run -d --name apache3 -p 127.0.0.1::8080 httpd        # Host-Port wird automatisch festgelegt
docker run -d --rm --name apache4 -p 8080 httpd              # Container port wird automatisch festgelegt
docker port <CID>                                            # Port-Mapping ausgeben

# Exkurs MYSQL
docker run --name mysql-1 -d -v /home/sascha/test:/var/lib/mysql/ \                     # Starten des mysql Containers
-p 13306:3306 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 \                            # Ports und Passwörter
-e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 mysql                           # DB-name
mysql -uuser1 -h 127.0.0.1  -pmypa55 -P13306 items < /home/sascha/sql_remove/db.sql     # (auf HOST-PC) SQL Datei vom Host in Container kopieren
bash-4.4# mysql -u user1 -p                                                             # (in Container)Login mysql als user1                                                 
mysql -uuser1 -h 127.0.0.1 -pmypa55  -P13306 items -e "SELECT * FROM Items"             # (auf HOST-PC) Abfrage über mysql client 
docker exec -it mysql-1  mysql -uuser1 -pmypa55 items -e "SELECT * FROM Items"          # SQL Abfrage über exec

# Zugriff auf Registrys
docker login registry.redhat.io # Nutzung von der Registry von redhat
docker search rhel # Suche in lokale/remote Repos nach verfügbaren Images
docker pull rhel   # Herunterladen eines Images
docker images      # Auflistung aller geladenen Images

# Ändern von Container-Images

docker rmi -a                                    # Löschen aller Images, die unbenutzt sind
docker rmi <Image>(:TAG)                         # Löscht komplettes Images oder nur ein Tag
docker diff <CID>                                # zeigt Änderungen Image -> Container
docker save -o httpd.tar custom-httpd:v1.0       # Lokale Ablage eines Images als tar (Alternative zu commit)
docker load -i httpd.tar                         # Wiederherstellen des Images
docker commit -a 'Sascha Hoffmann' \             # Erstellt neues Imaga auf Basis der Container Änderungen
official-httpd custom-httpd 
docker commit -c ‘CMD [“redis-server“]‘ <IMGID>  # Erzeugt ein Image aus einem laufenden Container mit einem Startkommando
docker tag custom-httpd:latest custom-httpd:v1.0 # Hinzufügen eines Tags


docker build -f Dockerfile.dev -t stephengrider/redis:latest .  # im aktuellen Ordner


# Weiter auf S. 80 /home/sascha/Dropbox/Dropbox_Sync/IT-Fortbildung/OpenShift/do180-4.10-student-guide.pdf


```
# Ändern von Container-Images -  Kapitel 4
```bash
# Speichern und Laden von Images in .tar Dateien
docker save -o httpd.tar httpd
docker load -i httpd.tar

# Löschen von Images im lokalen docker Storage

docker rmi
```
