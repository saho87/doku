# Git Befehle

```bash

# Hilfe
git --help                                      
git branch --help                               

# neues leeres Repository anlegen
git init
git clone [proj-Url]

# Änderungen aus Remote Branch abrufen
git pull
git pull origin main # Integration neuer Änderungen von main in aktuellen Branch (z.B. dev)

# Staging Area/ Index - Hinzufügen, Löschen, Umbenennen/Verschieben 
git add .
git add [file1] [file2]
git rm [file]
git mv [file] 

# Commit durchführen
git commit -m "Standard Commit durchführen"
git commit -am "Commit ohne add"

# Branch Mergen
git checkout main # Feature branch soll in main gemerged werden
git merge feature/update-readme

# Status von Arbeitsbereich und Staging Bereich anzeigen
git status

# git history - letzte Commits anzeigen
git log

# Unterschied von Arbeits- und Stagingbereich anzeigen
git diff [file]

# Unterschied von Stagingbereich und Repository anzeigen
git diff --staged [file]

# Dateien im Arbeitsverzeichnis wiederherstellen (wenn einfach nur gespeichert)
git restore [file]
git restore --source=[hash-commit] [file]
git checkout -- [file]

# Dateien im Staging Bereich wiederherstellen 
git restore --staged [hash-commit]
git restore --staged --source=4c3c03b69ec760ec16d4ccfeddc56b57c9c1ab4f [file]

# neuen Branch erstellen, pushen und löschen
git checkout -b [new-branch]
git push --set-upstream origin [new-branch]
git branch -d [deleted-branch]
git push origin -d [deleted-branch]

# Branch wechseln
git checkout main

# alle remote Branches anzeigen
git ls-remote

# Konfiguration initial einstellen
git config --global user.name "Sascha Hoffmann"
git config --global user.email "sascha.hoffmann@itzbund.de"

# Farbe verstellen
git config --global color.ui auto

# Remote Repository registrieren und testen
git remote set-url origin git@github.com:saho87/repository.git
git remote -v

# Probleme beim initialen Pushen großer Repos error: RPC failed; curl 56 HTTP/2 stream 5 was reset send-pack:
# unexpected disconnect while reading sideband packet fatal: the remote end hung up unexpectedly
git config --global http.version HTTP/1.1  # Ändern auf HTTP/1.1
git push
git config --global http.version HTTP/2    # Zurücksetzen auf HTTP/2

```
