##### Quelle: https://gitlab.com/nanuchi/youtube-tutorial-series/-/blob/master/basic-kubectl-commands/cli-commands.md?plain=0

###  Minikube Cluster erstellen
```bash
minikube start                  # start
kubectl get nodes
minikube status
kubectl version
kubectl cluster-info            # infos z.B. URL Cluster erhalten

```

### Cluster erstellen und in debug mode Neustarten
```bash

minikube delete
minikube start --v=7 --alsologtostderr
minikube status

```

### Ressourcen anlegen und bearbeiten
```bash
kubectl get nodes | pod | service | rs | deploy | event | svc (-o wide) --as user1
kubectl get all --selector app=App1,tier=frontend --no-headers -A (--all-namespaces)

# Objekte anlegen
kubectl run redis --image=redis --dry-run=client -o yaml --command -- sleep 1000 # Überschreibt ENTRYPOINT
kubectl run redis --image=redis -- --color red # ENRTYPOINT wird nicht geändert, nur ARGS hinzugefügt
kubectl create deployment my-dep --image=registry.k8s.io/echoserver:1.4 --replicas=3 --dry-run=client -o yaml
kubectl create -f service.yml # imperativ
kubectl create configmap sascha-cm --from-file=config.yaml # Dateiinhalt in CM einbinden
kubectl apply -f service.yml # deklarativ
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml # Service 1
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml # Service 2
kubectl create secret generic db-secret --from-literal DB_Host=sql01 

# Objekte ändern
kubectl edit deployment nginx-depl # imperativ, nicht empfohlen da yaml nicht persistiert
kubectl replace (--force)-f deployment.yaml # imperativ, empfohlen
kubectl apply -f service.yml # deklarativ, empfohlen
kubectl scale deployment nginx --replicas=5
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl set image deployment nginx nginx=nginx:1.18
kubectl delete pod nginx --force --grace-period 0 

# anderer Namespace
kubectl config set-context $(kubectl config current-context) --namespace=dev
kubectl get pods --n=default
kubectl get pods --all-namespaces

# taint und tolerance
kubectl taint nodes node-name key=value:taint-effect # NoSchedule | PreferNoSchedule | NoExecute

# Zusätzliche Befehle auf Pod ausführen 

kubectl run -it $POD_NAME --image=busibox -- sh
kubectl exec $POD_NAME -- env                                          
kubectl exec -it $POD_NAME -- bash                                      

# Erstellen eines Services Typ LoadBalancer

kubectl expose deployment hello-node --type=LoadBalancer --port=8080
# namespaced and cluster scoped resource types
kubectl api-resources --namespaced=true # listet alle Ressourcen inklusive Shortnames, apiVersion auf (namespacebasiert)
kubectl api-resources --namespaced=false
```


### debugging

```bash

kubectl logs {pod-name}
kubectl exec -it {pod-name} -- bash
kubectl describe service httpd-service

```
### kubeconfig
kubeclt config view --kubeconfig=my-kube-config # andere als die default konfig (~.kube/config) anschauen
kubeclt config --kubeconfig=my-kube-config use-context research # current config ändern
kubectl config set-context --current --namespace=default # aktuellen default namespace ändern
export KUBECONFIG=/root/my-kube-config # um config dauerhaft zu nutzen

### create or edit config file

```bash
vim nginx-deployment.yaml

# Ressource im Cluster aktualisieren (auch für das Hinzufügen neuer Ressourcen) -> bevorzugt

kubectl apply -f nginx-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml

# vorhandene Ressource wird durch neue Version ersetzt (Ressource ist ggf. kurzzeitig nicht verfügbar)

kubectl replace -f nginx-deployment.yaml
```


### Rollouts
```bash
kubectl rollout status deployment/my-deployment # Status
kubectl rollout history deployment/my-deployment # alle Revisions mit Grund anzeigen
kubectl rollout history deployment nginx-test --revision=2 # nur Infos von Revision 2
kubectl rollout undo deployment/my-deployment # vorheringes replicaset -> rollback 
kubectl rollout undo deployment/my-deployment --revision=1 # auf Revision 1 zurück
kubectl edit deployments nginx-test --record # fügt zusätzlich Revision in History mit Grund zu -> depricated
```

### OS Upgrades auf Nodes
```bash
kubectl drain node-1 # verlagert Pods von node-1 auf andere Nodes (müssen rs zugeordnet sein) + cordon
kubectl cordon node-2 # es können keine neuen Pods mehr auf node-2 ausgeführt werden
kubectl uncordon node-2 # es können wieder Pods auf node-2 ausgeführt werden
# Doku https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

### für Controlplane
# 1. *falls notwendig* Repo auf neue Major Version umstellen (z.B. von 1.29.x -> 1.30.x)
sudo apt-cache madison kubeadm # 2. welche Versionen von kubeadm können installiert werden?
# 3. Update von kubeadm
kubeadm version      # 4. kubeadm Version verifizieren
kubeadm upgrade plan # 5. Upgrade plan aufrufen (Welche Versionen sind verfügbar)
kubeadm upgrade apply v1.30.0 # 6.gewünschte Version installieren
# 7. Kubelet updaten

### für zusätzliche Nodes
# 1. *falls notwendig* Repo auf neue Major Version umstellen (z.B. von 1.29.x -> 1.30.x)
sudo apt-cache madison kubeadm # 2. welche Versionen von kubeadm können installiert werden?
# 3. Update von kubeadm
kubeadm version      # 4. kubeadm Version verifizieren
kubeadm upgrade node # 5. Worker node updaten
# 6. Kubelet updaten

```
### ETCD Backup and Restore
```bash
# Doku: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
# infos ETCD abholen
kubectl get pod etcd-controlplane -n kube-system -o yaml 
# Backup
ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db --endpoints 127.0.0.1:237
9 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kuberne
tes/pki/etcd/server.key
# Restore
ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup
# im etcd-pod volume auf neues data-dir ändern
vim /etc/kubernetes/manifests/etcd.yaml 
```
### Zertifikate
```bash

openssl genrsa -out sascha.key 2048 # Key erstellen
openssl req -new -key sascha.key -subj "/CN=sascha" -out sascha.csr # Signing Request erstellen
cat sascha.csr | base64 -w 0 # csr base64 codieren und in yaml CertificateSigningRequest Objekt einfügen
# der name im csr.object hat nichts mit dem Usernamen zu tun, Username wird mit openssl definiert (Subject: CN = sascha)
kubectl get csr
kubectl certificate approve|deny sascha # signing request bestätigen
kubectl get csr sascha -o yaml # den Teil unter "certificate" kopieren, decodieren und in sascha.crt einfügen
echo -n "LS0...Qo=" | base64 -d > sascha.crt
```

### API
```bash
kubectl proxy                  # Proxy (keine weitere Auth notwendig) starten um api-server zu erreichen
curl http://localhost:8080 -l  # Auth mit Zertifikat
    --key admin.key
    --cert admin.crt
    --cacert ca.crt
curl http://localhost:8080 -k # alle verfügbaren APIs anzeigen
curl .../apis/authentication.k8s.io # aktuelle und preferred Version einer API herausfinden
-runtime-config=rbac.authorization.k8s.io/v1alpha1 # in kube-apiserver.yaml hinzufügen um Entwicklerversion zu testen

# über Netzwerk
curl https://192.168.49.2:8443/api/v1/pods \
      --key /home/sascha/.minikube/sascha.key  \
      --cert /home/sascha/.minikube/sascha.crt \
      --cacert /home/sascha/.minikube/ca.crt
```

### RBAC
```bash
k auth can-i --list --as=system:serviceaccount:namespace1:default -n namespace2 # alle Rechte auflisten
k auth can-i create deployments --as user1 # prüfen ob user1 autorisiert ist...
k --as user1 get pod testpod # als user1 auf Ressource zugreifen

# A ClusterRole|Role defines a set of permissions and where it is available, in the whole cluster or just a single Namespace.
# A ClusterRoleBinding|RoleBinding connects a set of permissions with an account and defines where it is applied, in the whole cluster or just a single Namespace.

# Because of this there are 4 different RBAC combinations and 3 valid ones:
#    Role + RoleBinding (available in single Namespace, applied in single Namespace)
#    ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)
#    ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)
#    Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)
```

### Secrets

```bash
echo -n "admin" | base64
echo -n "YWRtaW4=" | base64 --decode
kubectl create secret docker-registry private-reg-cred \
    --docker-username=docker_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 \
    --docker-email=dock_user@myprivateregistry.com
kubectl create token <service-account-name> --duration=1h    # Token mit Ablaufdatum erstellen
jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64d | fromjson' <<< $TOKEN # Token zerlegen
# webseite für Zerlegen der Token:   https://jwt.io/
```

### Network

```bash
ip link        # devices
ip addr        # ip Adressen
ip route       # Routing 
cat /proc/sys/net/ipv4/ip_forward     # IP Weiterleitung true|false
arp            # Zuordnung MAC - IP
netstat -tulpn # offene NW-Verbindungen inkl. Ports

# /opt/cni/bin/   alle Network Plugins für Container Runtimes
# /etc/cni/net.d  welche Plugins werden genutzt

```

### Zusatz

```bash
# Anzeige der aktivierten Admission Control Plugins im API-Server
kubectl -n kube-system exec kube-apiserver-controlplane -- kube-apiserver -h | grep enable-admission-plugins

# Ausgabe der URL eines gewählten Services

minikube service mysql-service --url
 
# Service URL wird automatisch im Browser geöffnet bztw. nur die URL ausgegeben
minikube service hello-node
minikube service hello-node --url

# Addons Minikube verwalten

minikube addons list (alle anzeigen)
minikube addons disable metrics-server 
minikube addons enable dashboard

# Metric Server Monitoring aktivieren
minikube addons enable metrics-server
kubectl top node
kubectl top pod

# Dashboard

minicube dashboard 
  
# Proxy, der den Zugriff auf Pods ohne Service zulässt

kubectl proxy

# Podname in externer Variable speichern und auf Pod zugreifen

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo $POD_NAME
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/

Port forwarding
kubectl port-forward --namespace default service/code-server 8080:http

# Autocompletion, Alias
https://kubernetes.io/docs/reference/kubectl/quick-reference/
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.

alias k='kubectl'    # in .bashrc
complete -o default -F __start_kubectl k # # in .bashrc

cat <<EOF>> test
alias k='kubectl' 
complete -o default -F __start_kubectl k
EOF
```
### Status/Conditions von Pods
```bash
# 1. PodScheduled
Bedeutung: Der Pod wurde einem Node zugewiesen.
True: Der Pod wurde erfolgreich einem Node zugeordnet.
False: Der Pod wartet noch auf die Zuweisung eines Nodes (z. B. wegen fehlender Ressourcen).
Unknown: Der Status ist unklar (z. B. wegen Netzwerkproblemen oder Problemen mit dem Scheduler).

# 2. Initialized
Bedeutung: Alle Init-Container wurden erfolgreich ausgeführt.
Zustände:
True: Alle Init-Container wurden erfolgreich abgeschlossen.
False: Mindestens ein Init-Container läuft noch oder ist fehlgeschlagen.

# 3. PodReadyToStartContainers (ab Kubernetes 1.28)
Bedeutung: Der Pod ist bereit, Container zu starten, aber sie laufen noch nicht.
Zustände:
True: Der Pod hat alle Vorbereitungen getroffen (z. B. Volumes gemountet, Netzwerk konfiguriert).
False: Der Pod kann die Container noch nicht starten.

# 4. ContainersReady
Bedeutung: Alle Container im Pod sind gestartet und laufen ohne Probleme.
Zustände:
True: Alle Container sind im Zustand "running" und haben erfolgreich ihre Liveness-Probes (falls konfiguriert) bestanden.
False: Mindestens ein Container ist noch nicht bereit.

# 5. Ready
Bedeutung: Der Pod kann Traffic empfangen, Applikation ist gestartet
Zustände:
True: Alle Container sind bereit, haben ihre Readiness-Probes (falls vorhanden) bestanden und das Pod ist im Service-Routing aktiv.
False: Der Pod ist noch nicht bereit oder wurde aus dem Service-Traffic entfernt (z. B. wegen einer fehlerhaften Readiness-Probe).

```
  
