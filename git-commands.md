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
git commit --amend -m "Commit-Nachricht ändern"

# Branch Mergen
git checkout main # Feature branch soll in main gemerged werden
git merge feature/update-readme

# Status von Arbeitsbereich und Staging Bereich anzeigen
git status

# git history - letzte Commits anzeigen
git log branch
git log --oneline --graph # Commits in einer Zeile und grafisch
git log -- example.txt    # Commits die bestimmte Datei geändert haben
git log --pretty=format:"%h - %an, %ar : %s"


# Unterschied von Arbeits- und Stagingbereich anzeigen
git diff [file]

# Unterschied von Stagingbereich und Repository anzeigen
git diff --staged [file]

# Dateien im Arbeitsverzeichnis wiederherstellen (wenn einfach nur gespeichert)
git restore [file]
git restore --source=[hash-commit] [file]
git checkout -- [file]
git checkout HEAD~1 -- [file] # zeigt auf einen Commit vor HEAD

# Dateien im Staging Bereich wiederherstellen 
git restore --staged [hash-commit]
git restore --staged --source=4c3c03b69ec760ec16d4ccfeddc56b57c9c1ab4f [file]

# Commit rückgängig machen mit neuem Commit (nicht, wenn Änderungen schon remote gepushed wurden)
git revert [hash-commit]
git push

# Commits zurücksetzen
git reset --soft HEAD~1 # setzt HEAD-Zeiger um ein Commit zurück, Änderungen bleiben in Staging-Area
git reset --mixed c6e01150ca4c8d96bf969b06aa84e21a98ce4873 # wie soft, aber auch aus Staging Area entfernt
git reset --hard HEAD~1 # Rücksetzen von HEAD, Staging-Area, Arbeitsverzeichnis
git push --force # überschreibt den remote Branch

# Commits zurücksetzen mit rebase - Möglichkeit, mehrere Commits nachträglich zu bearbeiten
# pick: so lassen, reword: commit-Message ändern, edit: Inhalt ändern, squasch: mit vorherigen Commit zusammenführen, drop: löschen
git rebase -i [hash-commit] # interaktives Rebase öffnen
git push --force # überschreibt den remote Branch

# neuen Branch erstellen, pushen und löschen
git checkout -b [new-branch]
git push --set-upstream origin [new-branch]
git branch -d [deleted-branch]
git push origin -d [deleted-branch]

# Branch wechseln
git checkout main

# alle remote Branches anzeigen
git ls-remote

# alle Branches/Commits von remote holen
git fetch

# stash und unstash
git stash
git pop
git stash save "Beschreibung" 
git stash list # alle stashes anzeigen
git stash clear # alle stashes löschen
git stash drop # letzten stash löschen
git stash pop stash@{1}   # stash 1 anwenden und löschen
git stash apply            # letzten stash anwenden und beibehalten
git stash drop stash@{0} # stash 0 löschen

# interaktives Rebase für die letzten 3 Commits
git rebase -i HEAD~3
pick abc123 Commit 1
reword def456 Commit 2
pick ghi789 Commit 3

# Löschen von Commits
git reset --hard HEAD~2 # Löscht die letzten 2 Commits

# Konfiguration initial einstellen
git config --global user.name "Sascha Hoffmann"
git config --global user.email "sascha.hoffmann@itzbund.de"

# Farbe verstellen
git config --global color.ui auto

# wenn user/passwort beim Pushen verlangt werden muss von https auf ssh umgestellt werden
git remote -v                                                  # sollte mit https:// beginnen
git remote set-url origin git@github.com:saho87/repository.git # auf ssh umstellen
git remote -v                                                  # testen

# Probleme beim initialen Pushen großer Repos error: RPC failed; curl 56 HTTP/2 stream 5 was reset send-pack:
# unexpected disconnect while reading sideband packet fatal: the remote end hung up unexpectedly
git config --global http.version HTTP/1.1  # Ändern auf HTTP/1.1
git push
git config --global http.version HTTP/2    # Zurücksetzen auf HTTP/2

```
