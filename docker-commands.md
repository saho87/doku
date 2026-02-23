# Erstellen von containerisierten Services
```bash

# Zugriff auf man-Pages über man docker-(subcommand)
man docker-run

# Starten eines Containers -> Alternativ: run = create + start -a
# -it -Interaktiv, -d – im Hintergrund, --rm wird nach Ausführung gelöscht, --name – Vergabe eines Namens, -p- Portweiterleitung
docker run -d --rm --name my_container -p 8080:80 httpd:latest       # Portmapping 8080 des Hosts -> 80 des Containers
docker run -e GREET=Hello -e NAME=RedHat alpine  printenv GREET NAME # Nutzung von Umgebungsvariablen
-e MYSQL_PASSWORD=mypa55 -e MYSQL_DATABASE=items \
-e MYSQL_ROOT_PASSWORD=r00tpa55  \ -d registry.redhat.io/rhel8/mysql-80:1
docker run registry.redhat.io/rhel8/httpd-24 ls /tmp                 # container wird mit Kommando ls /tmp gestartet
docker restart my-httpd                                              # gestoppter Container wird neu gestartet
docker pause|unpause <CID>                                           # Container pausieren (SIGSTOP)

# Beenden von Containern
docker ps (-a)                                # aller laufenden (a- gelaufenen) Container ausgeben
docker stop <CID> (-a)   --time 20            # SIGTERM 20 sec grace period, dann SIGKILL (alle)
^stop^rm                                      # führt letzten Befehl mit rm anstatt von stop aus
docker kill (-s SIGKILL) <CID>                # SIGKILL, härten als stop
docker system prune                           # Löscht alle gestoppte Container
docker rm (-f)<CID>                           # Löscht einen gestoppten Container (-f stoppt ihn auch)

# Informationen zu Containern
docker logs <CID>            # Listet alle Logs zu einem Container auf
docker inspect <CID>         # detaillierte Infos zu Container
docker port <CID>         # genutzte Ports des Containers

# Prozess in bestehenden Container starten (-i - nimmt Eingaben entgegen -t pseudo Terminal zuweisen -e Umgebungsvariablen zuweisen -l letzten Container verwenden)
docker exec -it <CID> sh                 # Führt eine 2. Prozess in einem laufenden Container aus (hier öffnen einer shell)
docker exec -it <CID> /bin/bash -c 'ls'  # FÜhrt im Container ein Bash Commando aus
docker run -it <CID> /bin/bash           # Erstellt und führt neuen Container aus und gibt uns die Shell zum Zugriff
docker cp ./file 60ed77c57bb5:.          # Kopieren einer Datei vom Host in den Container

# Container-Netzwerke
docker network ls                                      # Anzeigen der Netzwerke
docker network inspect podman                          # Details zum Netzwerk anzeigen (z.B. subnet/IPs)
docker network create|rm -o isolate webapps            # neues (isoliertes) Netzwerk anlegen|löschen
docker run --net webapps                               # Container mit Netzwerk verbinden
docker inspect web1 | jq.[].NetworkSettings.Networks   # json Daten extrahieren
podman inspect --format='{{.State.Status}}' <CID>
docker network disconnect                              # Trennt einen Container von einem Netzwerk
docker network prun                                    # entfernt alle Netzwerke ohne Container
podman port -a | container                             # zeigt welche Ports verwendet werden

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
# podman Verzeichnis: /etc/containers/
docker login registry.redhat.io # Nutzung von der Registry von redhat
docker search rhel # Suche in lokale/remote Repos nach verfügbaren Images
docker pull rhel   --authfile $(AUTH_HARBOR) # Herunterladen eines Images
docker push registry.do180.lab:5000/httpd:twerks
docker push test-image:1.1 craas-09.itz.cloud.intranet.bund.de/import-itzbund-fms/test-image:1.1
docker images      # Auflistung aller geladenen Images

skopeo copy (--dest-tls-verify=false) \                # Image von Registry zu anderer Registry kopieren
docker://${RHOCP_REGISTRY}/default/python:3.9-ubi8 \
docker://registry.ocp4.example.com:8443/developer/python:3.9-ubi8

# Pushen von quay-io Repos mit Auth
# Anlegen eines neuen Repos in quay über Web-Gui z.B. duff-repo
# Berechtigen eines Users auf diesem Repos mit Schreibrechten
podman login -u="sascha_hoffmann+sascha_hoffmann_robot" -p="xxx" # quay.io login quay über Konsole
podman push quay.io/sascha_hoffmann/duff-nginx:1.0 # pushen des Repos

# Ändern von Container-Images 
# Ablage von Images unter /var/lib/docker/
docker rmi -a                                     # Löschen aller Images, die unbenutzt sind
docker rmi <Image>(:TAG)                          # Löscht komplettes Images oder nur ein Tag
docker diff <CID>                                 # zeigt Änderungen Image -> Container
docker save -o httpd.tar custom-httpd:v1.0        # Lokale Ablage eines Images als tar (Alternative zu commit)
docker export -o mytartfile.tar fb334n33434       # Export des Dateisystems eines Containers (Image Schichten/Metadata bleiben nicht erhalten)
docker load -i httpd.tar                          # Wiederherstellen des Images
docker import mytarfle.tar httpdcustom:2.4        # um einen tar-Container in ein Container-Image zu importieren
docker commit -a 'Sascha Hoffmann' \              # Erstellt neues Imaga auf Basis der Container Änderungen
official-httpd custom-httpd 
docker commit -c ‘CMD [“redis-server“]‘ <IMGID>   # Erzeugt ein Image aus einem laufenden Container mit einem Startkommando
docker tag custom-httpd:latest custom-httpd:v1.0  # Hinzufügen eines Tags zu einem bestehenden Image
docker image prune -a                             # löschen aller nicht als Container ausgeführten Images

# Erstellen von Dockerfiles/Containerfiles
man Containerfile # Doku
   # This is a comment line                                    # Kommentar
   FROM ubi8/ubi:8.5                                           # Basisimage
   LABEL description="This is a custom httpd container image"  # Metadaten hinzufügen (kein zusätzlicher Layer)
   MAINTAINER John Doe <jdoe@xyz.com>                          # Autor (auch Metadaten)
   RUN yum install -y httpd && \                               # y (nicht interaktiv)
       yum clean                                               # && (nur weiter wenn vorheriges Kommando erfolgreich)
                                                               # clean reduziert Image Größe
   WORKDIR /custom                                             # legt das Arbeitsverzeichnis im Container fest
   EXPOSE 80                                                   # zeigt an, dass Container Port überwacht (nur Metadaten)
   ENV LogLevel "info" \                                       # Umgebungsvariablen
       MYSQL_DATABASE="my_database"
   ARG test="1.2.3"                                            # Variablen, die nur beim Bau des Images verfügbar sind, nicht später im Container
   ADD http://someserver.com/archive.tar /var/www/html         # kopiert + extrahiert Files aus lok./remote Quelle in Container-FS
   COPY ./src/ /var/www/html/                                  # kopiert Dateien rel. zu Arbeitsverzeichnis in Container-FS
   USER apache                                                 # User/UID für Verwendung von RUN, CMD oder ENTRYPOINT
                                                               # Openshift ignoriert USER-ID und legt Random User an
                                                               # zusätzlich fügt Openshift den User zur Gruppe Root hinzu
   ENTRYPOINT ["sleep"]                                        # Standardbefehl zur Ausführung des Containers -> command (kubernetes)
   CMD ["10"]                                                  # Standardargumente für ENTRYPOINT --> args (kubernetes)

# Bauen von Images
podman build -f Containerfile -t simple-server                 # Angabe der Dockerfile und Taggen des Images
podman image tree <Image>                                      # visualiert Layer, Image+Layer-Größe, Tags usw.

# Unterschied ENTRYPOINT/CMD
ENTRYPOINT kann erweitert werden und kann Parameter von run <image> <Erweiterung> annehmen, die hier "10" überschreiben
docker run <image> 20                                  # Überschreibt nur CMD
docker run --entrypoint sleep2.0 <image> 50            # Überschreibt kompletten Entrypoint zur Laufzeit + CMD (am Ende)
ENTRYPOINT entspricht in Kubernetes command
CMD entspricht in Kubernetes args

# systemd service aus Container generieren
sudo podman generate systemd reg-httpd | sudo tee -a /usr/lib/systemd/system/reg-httpd.service

# Docker network
docker network ls             # alle NW-Verbindungen anzeigen
docker network inspect bridge # Details wie IP, inferface name
```
# Skopeo
```bash

skopeo list-tags docker://registry.do180.lab:5000/httpd  # alle Tags eine Images anzeigen
skopeo inspect docker://registry.do180.lab:5000/httpd    # Details zu einem Image anzeigen
skopeo copy docker://registry1.do180.lab:5000/httpd docker://registry2.do180.lab:5000/httpd # Kopie von Reg1 zu Reg2

```

# Nexus
```bash
# Ordner erstellen und Ownership ändern
sudo mkdir /home/sascha/Projects/nexus/nexus-data/  && chown -R 200 /home/sascha/Projects/nexus/nexus-data/
# Anwendung in Docker starten
docker run -d -p 8081:8081 --name nexus -v /home/sascha/Projects/nexus/nexus-data:/sonatype-work sonatype/nexus
# Aufruf webbrowser / admin|admin123
http://localhost:8081/nexus/
```
