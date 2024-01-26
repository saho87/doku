Doku local cluster: https://www.redhat.com/sysadmin/install-openshift-local
Probleme wildcard: https://askubuntu.com/questions/1029882/how-can-i-set-up-local-wildcard-127-0-0-1-domain-resolution-on-18-04-20-04/1031896#1031896
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

# neue App aus php Code in Git Repo erstellen (S2I-Prozess /Source-to-Image
oc new-app php:7.3 --name=php-helloworld https://github.com/saho87/DO180-apps#s2i --context-dir php-helloworld


# weiter auf S. 201 /home/sascha/Dropbox/Dropbox_Sync/IT-Fortbildung/OpenShift/do180-4.10-student-guide.pdf

```
# OpenShift local Cluster einrichten
```bash
# Doku local cluster: https://www.redhat.com/sysadmin/install-openshift-local
# wild-card cert:https://askubuntu.com/questions/1029882/how-can-i-set-up-local-wildcard-127-0-0-1-domain-resolution-on-18-04-20-04/1031896#1031896
echo 'address=/.apps-crc.testing/192.168.130.11' | sudo tee /etc/NetworkManager/dnsmasq.d/apps-crc.testing-wildcard.conf

```
