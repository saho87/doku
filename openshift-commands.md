Doku local cluster: https://www.redhat.com/sysadmin/install-openshift-local
# CRC-commands (Local Cluster)
```bash
oc login <clusterURL>  # Login für console
oc api-resources # Liste aller verfügbarenen Ressourcen und Abkürzungen

oc get <RES_TYPE>                 # Infos zu allen Ressourcen eines Typs
oc get service mysql -o yaml      # Infos zu einer Ressource in yaml
oc get all                        # alle Ressourcentypen
oc describe <RES_TYPE> <RES_NAME> # Details zu Ressource
oc create                         # Erstellen von Ressourcen aus ResDefinitionen
oc edit                           # Bearbeiten von ResDefinitionen (öffnen vi)
oc delete <RES_TYPE> <RES_NAME>
oc exec <CID>                     # Ausführung von Befehlen in Container

oc port-forward <CID> 3306:3306 # Host Port 3306 -> DB Port 3306
oc new-app mysql -o yaml # Vorlagendatei für Manifest in yaml oder auch json
oc new-app mysql -o yaml MYSQL_USER=user \         
MYSQL_PASSWORD=pass MYSQL_DATABASE=testdb -l db=mysql
oc create -f pv1001.yaml  # erstellt eine Ressource auf Basis eine abgelegten yaml
oc status # Status und ob Deployment erfolgreich

# weiter auf S. 166 /home/sascha/Dropbox/Dropbox_Sync/IT-Fortbildung/OpenShift/do180-4.10-student-guide.pdf

```
