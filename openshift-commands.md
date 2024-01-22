Doku local cluster: https://www.redhat.com/sysadmin/install-openshift-local
# CRC-commands (Local Cluster)
```bash
oc login <clusterURL>  # Login für console
oc api-resources # Liste aller verfügbarenen Ressourcen und Abkürzungen

oc port-forward mysql-openshift-1-glqrp 3306:3306 # Host Port 3306 -> DB Port 3306
oc new-app mysql -o yaml # Vorlagendatei für Manifest in yaml oder auch json
oc new-app mysql -o yaml MYSQL_USER=user \         
MYSQL_PASSWORD=pass MYSQL_DATABASE=testdb -l db=mysql

```
