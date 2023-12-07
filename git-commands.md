# Git Befehle

```bash

git --help                                      # Hilfe
git branch --help                               

##### neues leeres Repository anlegen
git init
git clone [proj-Url]

##### Staging Area/ Index - Hinzufügen, Löschen, Umbenennen/Verschieben 
git add *
git add [file]
git rm [file]
git mv [file] 

##### Commit durchführen
git commit -m "Standard Commit durchführen"
git commit -am "Commit ohne add"

##### Status von Arbeitsbereich und Staging Bereich anzeigen
git status

##### git history - letzte Commits anzeigen
git log

##### Unterschied von Arbeits- und Stagingbereich anzeigen
git diff [file]

##### Unterschied von Stagingbereich und Repository anzeigen
git diff --staged [file]

##### Dateien im Arbeitsverzeichnis wiederherstellen (wenn einfach nur gespeichert)
git restore [file]
git restore --source=[hash-commit] [file]
git checkout -- [file]

##### Dateien im Staging Bereich wiederherstellen 
git restore --staged [hash-commit]
git restore --staged --source=4c3c03b69ec760ec16d4ccfeddc56b57c9c1ab4f [file]

##### neuen Branch erstellen, pushen und löschen
git checkout -b [new-branch]
git push --set-upstream origin [new-branch]
git branch -d [new-branch]

##### Branch wechseln
git checkout main

##### Konfiguration initial einstellen
git config --global user.name "Sascha Hoffmann"
git config --global user.email "sascha.hoffmann@itzbund.de"

##### Farbe verstellen
git config --global color.ui auto
```