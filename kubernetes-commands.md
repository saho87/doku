##### Quelle: https://gitlab.com/nanuchi/youtube-tutorial-series/-/blob/master/basic-kubectl-commands/cli-commands.md?plain=0

###  Minikube Cluster erstellen
```bash
minikube start                  # start
kubectl get nodes
minikube status
kubectl version
kubectl cluster-info

```

&nbsp;

### Cluster erstellen und in debug mode Neustarten
```bash

minikube delete
minikube start --v=7 --alsologtostderr
minikube status

```
&nbsp;

### Ressourcen anlegen und bearbeiten
```bash
kubectl get nodes | pod | service | rs | deploy | event | svc

# Objekte anlegen
kubectl run redis --image=redis 
kubectl create deployment my-dep --image=registry.k8s.io/echoserver:1.4 --replicas=3 --dry-run=client -o yaml
kubectl create -f service.yml # imperativ
kubectl apply -f service.yml # deklarativ
kubectl expose deployment nginx --port 80

#Objekte ändern
kubectl edit deployment nginx-depl # imperativ, nicht empfohlen da yaml nicht persistiert
kubectl replace -f deployment.yaml # imperativ, empfohlen
kubectl apply -f service.yml # deklarativ, empfohlen
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18

# anderer Namespace
kubectl config set-context $(kubectl config current-context) --namespace=dev
kubectl get pods --n=default
kubectl get pods --all-namespaces

# Zusätzliche Befehle auf Pod ausführen 

kubectl exec $POD_NAME -- env                                          
kubectl exec -it $POD_NAME -- bash                                      

# Erstellen eines Services Typ LoadBalancer

kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```
&nbsp;

### debugging

```bash

kubectl logs {pod-name}
kubectl exec -it {pod-name} -- bin/bash
kubectl describe service httpd-service

```
&nbsp;

### create or edit config file

```bash

vim nginx-deployment.yaml

# Ressource im Cluster aktualisieren (auch für das Hinzufügen neuer Ressourcen) -> bevorzugt

kubectl apply -f nginx-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml

# vorhandene Ressource wird durch neue Version ersetzt (Ressource ist ggf. kurzzeitig nicht verfügbar)

kubectl replace -f nginx-deployment.yaml

```
&nbsp;

### Zusatz

```bash
# Ausgabe der URL eines gewählten Services

minikube service mysql-service --url
 
# Service URL wird automatisch im Browser geöffnet bztw. nur die URL ausgegeben
minikube service hello-node
minikube service hello-node --url

# Addons Minikube verwalten

minikube addons list (alle anzeigen)
minikube addons disable metrics-server 
minikube addons enable dashboard

# Dashboard

minicube dashboard 
  
# Proxy, der den Zugriff auf Pods ohne Service zulässt

kubectl proxy

# Podname in externer Variable speichern und auf Pod zugreifen

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo $POD_NAME
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/
```
  
