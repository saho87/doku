# Erstellen von containerisierten Services
```bash
docker search rhel # Suche in lokale/remote Repos nach verfügbaren Images
docker pull rhel   # Herunterladen eines Images
docker images      # Auflistung aller geladenen Images

# Starten eines Containers -> Alternativ: run = create + start -a
# it -Interaktiv, d – im Hintergrund, rm wird nach Ausführung gelöscht, name – Vergabe eines Namens, p- Portweiterleitung
docker run -d --rm --name my_container -p 8080:80 registry.access.redhat.com/ubi8:latest # Portmapping 8080 des Hosts -> 80 des Containers
docker run -e GREET=Hello -e NAME=RedHat alpine  printenv GREET NAME # Nutzung von Umgebungsvariablen
docker run --rm --name mysql-basic  -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55  \ # Erstellen einer mysql-DB mit Credentials
-e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55  \
-d registry.redhat.io/rhel8/mysql-80:1

docker port <Container-NR>  # Port-Mapping ausgeben 
docker ps (-a)  # aller laufenden (a- gelaufenen) Container ausgeben

# Beenden von Containern
docker kill    # SIGTERM/SIGKILL Nachricht wird an den laufenden Prozess gesendet
docker stop    # (bevorzugt, könnte aber länger dauern)
docker system prune  # Löscht gestoppte container

docker logs <id>  # Listet alle Logs zu einem Container auf

docker exec -it 1691237dd98e sh    # Führt eine 2. Prozess in einem laufenden Container aus (hier öffnen einer shell)
docker run -it busybox /bin/bash   # Erstellt und führt neuen Container aus und gibt uns die Shell zum Zugriff 

# Bau von Images mit (f)ilename und (t)ag
docker build -f Dockerfile.dev -t stephengrider/redis:latest .  # im aktuellen Ordner
docker login registry.redhat.io # Nutzung von der Registry von redhat
# Erzeugt ein Image aus einem laufenden Container mit einem Startkommando
Docker commit -c ‘CMD [“redis-server“]‘ <IMGID>

```
# Ändern von Container-Images -  Kapitel 4
```bash
# Speichern und Laden von Images in .tar Dateien
docker save -o httpd.tar httpd
docker load -i httpd.tar

# Löschen von Images im lokalen docker Storage

docker rmi
```
