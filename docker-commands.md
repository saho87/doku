# Starten eines Containers
# it -Interaktiv, d – im Hintergrund, rm wird nach Ausführung gelöscht, name – Vergabe eines Namens, p- Portweiterleitung
# Alternativ: run = create + start -a
Docker run -it -d --rm --name -p 8080:80 my_container httpd /bin/bash

# Gibt eine Liste aller laufenden (a- gelaufenen) Container aus
Docker ps (-a)


# SIGTERM/SIGKILL Nachricht wird an den laufenden Prozess gesendet - akzeptiert Namen und IDs
Docker kill
Docker stop (bevorzugt, könnte aber länger dauern)

# Listet alle Logs zu einem Container auf
Docker logs <id>

# Löscht gestoppte container
Docker system prune

# Führt eine 2. Prozess in einem laufenden Container aus (hier öffnen einer shell)
# -i (SDIN) -t (formatierte Ausgabe und mehr)
docker exec -it 1691237dd98e sh

# Erstellt und führt neuen Container aus und gibt uns die Shell zum Zugriff (auch /bin/bash möglich)
sudo docker run -it busybox sh

# Image wird angefertigt im aktuellen Ordner
# Angabe der f- Dockerfile und eines t- tags
docker build -f Dockerfile.dev -t stephengrider/redis:latest .

# Erzeugt ein Image aus einem laufenden Container mit einem Startkommando
Docker commit -c ‘CMD [“redis-server“]‘ <IMGID>


# ############################## Ändern von Container-Images -  Kapitel 4 ########################################

# Speichern und Laden von Images in .tar Dateien
docker save -o httpd.tar httpd
docker load -i httpd.tar

# Löschen von Images im lokalen docker Storage

docker rmi
