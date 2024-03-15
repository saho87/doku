e# Login DO288
```bash
oc login -u developer -p developer https://console-openshift-console.apps.ocp4.example.com:6443
oc login -u admin -p redhatocp https://console-openshift-console.apps.ocp4.example.com:6443
podman login -u="developer" -p="HCjhSca2LgddivwRAUazqBcatBMKWDSos57ZLfQyazWYK6tiYfgnER4gP0X5/IxR" registry.ocp4.example.com:8443
https://meet.google.com/kmp-ybuo-nbb?pli=1 
```
# Doku
```bash
Doku local cluster: 
https://www.redhat.com/sysadmin/install-openshift-local

Doku open-shift platform:
https://docs.openshift.com/container-platform/4.14/welcome/index.html

Probleme wildcard: 
https://askubuntu.com/questions/1029882/how-can-i-set-up-local-wildcard-127-0-0-1-domain-resolution-on-18-04-20-04/1031896#1031896

Doku DO180 von anderem User:
https://github.com/fahmifahim/openshift/tree/master

exam: https://github.com/FWSquatch/do180-practice
```

# CRC-commands (Local Cluster)
```bash
oc login <clusterURL>  # Login für console
oc login -u ${RHT_OCP4_DEV_USER} -p  ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
oc whoami --show-console          # URL Console anzeigen
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
oc project sascha-ocp             # Namespace wechseln


oc new-project sascha-mysql # legt neuen Namespace an und wechselt dorthin
oc create deployment test \  # erstellt Deployment auf Basis von Image
--image registry.ocp4.example.com:8443/redhattraining/hello-world-nginx
oc new-app mysql -o yaml MYSQL_USER=user \         
MYSQL_PASSWORD=pass MYSQL_DATABASE=testdb -l db=mysql
--dry-run -o yaml        # keine Ressourcen werden erstell, aber Ausgabe als yaml
oc new-app -i mysql      # oc get is mysql -n openshift
oc create -f pv1001.yaml  # erstellt eine Ressource auf Basis eine abgelegten yaml

oc status # Status und ob Deployment erfolgreich
oc delete all --selector app=greetings # app + Ressourcen entfernen
oc delete all --all  # alle Ressourcen löschen

# Routen (Verbindung zw. öffentlichen IP und DNS-Hostname zu interner Service-IP)
oc expose service quotedb --name quote  # Route über oc zu Service erstellen
oc create route edge todo-https \       # route mit edge
--service todo-http --hostname todo-https.apps.ocp4.example.com
oc get pod --all-namespaces | grep router # Standard Routing-Service abrufen
oc port-forward <CID> 3306:3306 # Alternativ: Portweiterleitung direkt in Pod

oc logs -f <CID> # fortlaufende Logs

# S2I: neue App aus php Code aus Git Repo erstellen (S2I-Prozess /Source-to-Image) Branch ist s2i
oc new-app <Img-Stream> --name=<AppName> <Repo#Branch> --centext-dir <Dir>
oc new-app php:7.3 --name=php-helloworld https://github.com/saho87/DO180-apps#s2i --context-dir php-helloworld
# neuen Build-Prozess starten (nach Änderung)
oc start-build php-helloworld

# Umgebungsvariablen, die allen Pods nach Erstellung eines mysql Services zur Verfügung stehen:
MYSQL_SERVICE_HOST=10.0.0.11
MYSQL_SERVICE_PORT=3306
MYSQL_PORT=tcp://10.0.0.11:3306
MYSQL_PORT_3306_TCP=tcp://10.0.0.11:3306
MYSQL_PORT_3306_TCP_PROTO=tcp
MYSQL_PORT_3306_TCP_PORT=3306
MYSQL_PORT_3306_TCP_ADDR=10.0.0.11

# weiter auf S. 201 /home/sascha/Dropbox/Dropbox_Sync/IT-Fortbildung/OpenShift/do180-4.10-student-guide.pdf

```
# OpenShift local Cluster einrichten
```bash
# Doku local cluster: https://www.redhat.com/sysadmin/install-openshift-local
# wild-card cert:https://askubuntu.com/questions/1029882/how-can-i-set-up-local-wildcard-127-0-0-1-domain-resolution-on-18-04-20-04/1031896#1031896
echo 'address=/.apps-crc.testing/192.168.130.11' | sudo tee /etc/NetworkManager/dnsmasq.d/apps-crc.testing-wildcard.conf

```

# DO288
```bash
# Todo
- Was ist ein Service Account, was kann man damit machen? Insbesonder Pull SEcrets
- Was hat Rolebinding damit zu tun?
- Was bedeutet ROlling Update, welche anderen Strategien gibt es?
- Was bedeutet RWO und RWOP? https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
- Was macht Istio (Service Mesh)
- Security Context Constraints
- Was hat SEcurity mit labels zu tun (oc describe project)
- Wann brauche ich ein stateful set?
```
```bash
vi .kube/config  # config Cluster

# Secrets
oc explain sa
oc describe sa default
oc secrets link --for=pull default training-registry     # fügt neues Image Pull Secret dem SA hinzu
oc secrets unlink default training-registry              # entfernt Image Pull Secret dem SA
oc secrets link --for=mount default training-registry    # fügt neues Secrets für das mounten eines Containers
oc set env deploy/expense-service \                      # Umgebungsvariablen aus bestehenden Secrets setzen
--from=secret/postgresql  
oc create secret docker-registry SECRET_NAME \           # neues Secret über Kommandozeile
--docker-server REGISTRY_URL \
--docker-username USER \
--docker-password PASSWORD \
--docker-email=EMAIL
oc extract secret/postgresql --to=.                      # Secret in aktuellen Ordner extrahieren

# Debugging
oc debug -t deployment/todo-http \                       # Debugging-Pod starten
--image registry.ocp4.example.com:8443/ubi8/ubi:8.4
oc rsh pod123                                            # Console auf Pod starten

# Builder Images
oc new-app --name java-application \
  --build-env BUILD_ENV=BUILD_VALUE \                          # env für Pods
  --strategy                                                   # 
  --env RUNTIME_ENV=RUNTIME_VALUE \                            # env für Runtime-Pods
  -i redhat-openjdk18-openshift:1.8 \                          # image stream Builder Image
  --context-dir java-application \                             # Anwendungsverzeichnis in Git
  https://git.example.com/example/java-application-repository  # Speicherort Git

# Beispiel aus 4.4 DO288
oc new-app --name vertx-site \
--build-env \
MAVEN_MIRROR_URL=http://nexus-infra.apps.ocp4.example.com/java \
--env JAVA_APP_JAR=vertx-site-1.0.0-SNAPSHOT-fat.jar \
-i redhat-openjdk18-openshift:1.8 \
--context-dir apps/builds-applications/vertx-site \
https://git.ocp4.example.com/developer/DO288-apps

oc start-build java-appliction             # 2. Build ausführen
oc start-build --follow bc/vertx-site      # 2. Build mit logs Ausgabe starten
oc cancel-build bc/java-application        # Build abbrechen
oc set env bc/java-application BUILD_LOGLEVEL=3
oc wait --for=condition=complete \         # auf Fertigstellung des Builds warten
  --timeout=600s build/vertx-site-1
oc patch deployment hello -p \             # Aktualisieren oder Hinzufügen von Feldern 
'{"spec":{"template":{"spec":{"resources":{"requests":{"cpu": "100m"}}}}}}'

# Image Stream 
oc import-image custom-server --confirm \    # neuer Image Stream
--from registry.ocp4.example.com:8443/developer/custom-server:1.0.0
oc new-app --name custom-server \            # Application aus Image Stream
-i images-review/custom-server

# Webhook Trigger
oc set triggers bc/name --from-image=project/image:tag         # setzen von Triggern
oc set triggers deploy/hello \                                 # Trigger auf Deployment
  --from-image=hello:latest -c hello
oc set triggers deploy/hello                                   # Trigger anzeigen
oc tag \                                                       # neuen Tag setzen
  registry.ocp4.example.com:8443/redhattraining/ocpdev-builds-triggers-hello:v2 \
  hello:latest

# Deployment Strategien
https://docs.openshift.com/gitops/1.11/argo_rollouts/using-argo-rollouts-for-progressive-deployment-delivery.html
https://argoproj.github.io/argo-rollouts/features/specification/

# Config maps
# Doku: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
oc create cm init-db-cm --from-file init-db.sql # cm von Datei erstellen

# Security
# https://kubernetes.io/docs/concepts/security/pod-security-admission/

# Metadaten unprivelegiert beschaffen: https://kubernetes.io/docs/concepts/workloads/pods/downward-api/

# Persistent-Volume-Claims (PVC) und Volumes

# Limits/Requests https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
# Memory: Limit = Request -> gleich dimensionieren
# CPU: Limit > Requests
oc api-resources --api-group="" --namespaced=true    # alle Core-Ressourcen auflisten
oc create resourcequota example --hard=count/pods=1  # Anzahl der Pods im Namespace auf 1 begrenzen
oc get quota
oc get limits
oc set resources deployment test --requests=cpu=1
oc adm top node            # Ressourcennutzung der Nodes anzeigen
oc describe node/master01  # Ressourcen eines Nodes anzeigen

# Clusterresources:
oc create clusterresourcequota example               # Clusterresourcequota -> limit CPU auf 10
--project-label-selector=group=dev --hard=requests.cpu=10
oc describe AppliedClusterResourceQuota -n example-2 # falls keine Leserechte auf ClusterResourceQuotas

# Helm
Helm repo list                                               # verbundene Repos anzeigen
helm repo add bitnami https://charts.bitnami.com/bitnami     # neues Repo hinzufügen
helm search repo mariadb                                     # in vorhandenen Repos nach mariadb suchen
helm create demoapp
helm pull bitnami/mariadb
helm install quarkus-demo openshift/redhat-quarkus           # helm charts installieren (nur im Cluster)
helm template . (wie oc process)
helm template -s templates/deployment .
Helm install apache-demo .
Helm uninstall apache-demo
helm upgrade apache-demo .
helm template -s templates/postgres-secret.yaml . \             # Ressource aus helm template erzeugen
helm template -s templates/postgres-secret.yaml . \             # Ressourcen direct in Cluster deployen
  oc apply -f f --dry-run=client --validate
# Kustomize
touch kustomization.yaml     # angeben, welche Dateien ich benutzen werden
resources:
  - pvc-apache.yaml
  - www.data-cm.yaml
  - deployment.yaml
oc apply -k .

# Prometheus
https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/

# Service-Accounts / Rolebinding / Clusterrolebinding
oc create sa sascha-sa   # SA für aktuellen Namespace erstellen
oc adm policy add-scc-to-user anyuid -z sascha-sa    # Clusterrolebinding erstellen
oc create clusterrolebinding sascha-crb --clusterrole system:openshift:scc:anyuid --user sascha-sa  # CRB erstellen 2. Möglichkeit
oc create rolebinding app-team --clusterrole edit --group app-team
oc set sa deployment/nginx sascha-sa  # Service-Account dem Deployment zuweisen
```

# DO280
```bash
# User Management
oc get oauth cluster \                                   # config identity provider lokal ablegen -> ändern
-o yaml > ~/DO280/labs/auth-providers/oauth.yaml
oc replace -f ~/DO280/labs/auth-providers/oauth.yaml     # geänderte config in CLuster hochladen
oc extract secret/htpasswd-secret -n openshift-config \  # aus Secret eine Passwort Datei extrahieren
--to /tmp/ --confirm
htpasswd -D /tmp/htpasswd manager                        # User löschen (auch manuell in Datei möglich)
htpasswd -b ~/DO280/labs/auth-providers/htpasswd \       # User hinzufügen
manager redhat
oc set data secret/htpasswd-secret \                     # Secret updaten
--from-file htpasswd=/tmp/htpasswd -n openshift-config
oc get identities                                        # Identity aus Open-Shift auslesen
oc delete user manager                                   # Ressourcen von User löschen
htpasswd -n -b dba redhat                                # erstellt Hash -> Ausgabe Konsole
oc create secret generic htpasswd-secret \
--from-file htpasswd=/tmp/htpasswd -n openshift-config
oc adm policy add-cluster-role-to-user cluster-admin student # Clusterrechte einem User zuweisen
oc adm groups new developers                             # neue Gruppe erstellen

# RBAC
oc adm policy add-cluster-role-to-user {cluster-role} {username}       # Cluster-Role zu User hinzufügen
oc adm policy remove-cluster-role-from-user {cluster-role} {username}  # Cluster-Role von User löschen
oc adm policy who-can delete user                                      # Herausfinden ob user Befehl ausführen kann
oc policy add-role-to-user {role-name} {username} -n {namespace}       # Rolle zu User hinzufügen

# Network Security
https://www.redhat.com/architect/encryption-secure-routes-openshift

# Network Policies
# Beispiel yaml:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: 'allow-specific'
  namespace: network-policy
spec:
  podSelector:
    matchLabels:
      deployment: hello  # Zielpods, wenn leer, dann alle Pods im Namespace
  ingress:               # Liste von ingress traffic rules
    - from:
      - namespaceSelector:  # Regel -> welche Namespaces
          matchLabels:
            network: different-namespace # muss durch label explizit dem Namespace hinzugefügt werden
        podSelector:  # Regel -> welche Pods
          matchLabels:
            deployment: sample-app # 
      ports:
      - port: 8080
        protocol: TCP
# interner Traffic mit TLS
oc annotate service hello \                    # Generieren eines Zertifikats und Schlüsselpaares für einen Service
service.beta.openshift.io/serving-cert-secret-name=hello-secret

oc label namespace different-namespace \ # Namespace muss extra gelabelt werden
network=different-namespace

# Selfservice und Templating

oc edit clusterrolebinding self-provisioners        # dann subject ändern (z.B. neue Gruppe)
oc annotate clusterrolebinding/self-provisioners \  # verhindert das Wiederherstellen des orig. CRB bei Cluster Neustart
--overwrite rbac.authorization.kubernetes.io/autoupdate=false
oc adm create-bootstrap-project-template \          # initiales Projekt Template anlegen (dann modifizieren)
-o yaml >template.yaml
oc create -f template.yaml -n openshift-config      # Template in open-shift erstellen
oc edit projects.config.openshift.io cluster        # cluster project config ändern
watch oc get pod -n openshift-apiserver             # api server pods beobachten

# Security Context Constraints (SCCs)
oc get scc
oc describe scc anyuid
oc describe pod {podnam} | grep scc                 # nicht jeder Pod hat scc
oc get pod podname -o yaml | \
oc adm policy scc-subject-review -f -
oc create serviceaccount service-account-name        # SA wird zur Änderung einer SCC des Containers benötigt
oc adm policy add-scc-to-user anyuid -z service-account # SA wird an SCC geknüpft
oc set serviceaccount deployment/deployment-name \   # SA einem Deployment zuweisen, z.B. wenn Pod keine REchte
service-account-name                                 # ein Verzeichnis zu erstellen

# Applikationen Zugang zur Kubernetes API erlauben
# Rolle anlegen oder Default OpenShift Role (z.B. edit) nutzen
oc adm policy add-role-to-user cluster-role -z service-account       # Rolle an SA binden  (nur Namespace)
oc adm policy add-cluster-role-to-user cluster-role service-account  # Rolle an SA binden (Clusterweit)
# SA den Pods zuweisen
system:serviceaccount:project:service-account # von anderen Projekte auf SA referenzieren
   
# Images referenzieren
- quay.io/sascha_hoffmann/apache:1.2                     # Tag verwenden
- quay.io/sascha_hoffmann/apache@sha256:4578...          # Hash verwerden Vorteil: eindeutig
- quay.io/sascha_hoffmann/apache:1.2-alpha.1             # best practice: eindeutige Build Nummer verwenden
```
