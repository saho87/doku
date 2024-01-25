Doku local cluster: https://www.redhat.com/sysadmin/install-openshift-local
# CRC-commands (Local Cluster)
```bash
oc login <clusterURL>  # Login für console
oc login -u ${RHT_OCP4_DEV_USER} -p  ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
# Ressourcen anlegen und löschen
oc api-resources # Liste aller verfügbarenen Ressourcen und Abkürzungen
oc get <RES_TYPE> (-w)            # Infos zu allen Ressourcen eines Typs (w überwacht)
oc get service mysql -o yaml      # Infos zu einer Ressource in yaml
oc get all                        # alle Ressourcentypen
oc describe <RES_TYPE> <RES_NAME> # Details zu Ressource
oc create                         # Erstellen von Ressourcen aus ResDefinitionen
oc edit                           # Bearbeiten von ResDefinitionen (öffnen vi)
oc delete <RES_TYPE> <RES_NAME>
oc exec <CID>                     # Ausführung von Befehlen in Container


oc new-project sascha-mysql # legt neuen Namespace an und wechselt dorthin
oc new-app mysql -o yaml # Vorlagendatei für Manifest in yaml oder auch json
oc new-app mysql -o yaml MYSQL_USER=user \         
MYSQL_PASSWORD=pass MYSQL_DATABASE=testdb -l db=mysql
oc create -f pv1001.yaml  # erstellt eine Ressource auf Basis eine abgelegten yaml
oc status # Status und ob Deployment erfolgreich

# Routen
oc expose service quotedb --name quote  # Route über oc zu Service erstellen
oc get pod --all-namespaces | grep router # Standard Routing-Service abrufen
oc port-forward <CID> 3306:3306 # Alternativ: Portweiterleitung direkt in Pod

oc logs -f <CID> # fortlaufende Logs


# weiter auf S. 186 /home/sascha/Dropbox/Dropbox_Sync/IT-Fortbildung/OpenShift/do180-4.10-student-guide.pdf

```
