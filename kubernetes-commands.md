### create minikube cluster
`minikube start`

`kubectl get nodes`

`minikube status`

`kubectl version`

### delete cluster and restart in debug mode

`minikube delete`

`minikube start --vm-driver=hyperkit --v=7 --alsologtostderr`

`minikube status`

### kubectl commands
`kubectl get nodes`

`kubectl get pod`

`kubectl get services`

`kubectl create deployment hello-node --image=registry.k8s.io/echoserver:1.4`

`kubectl get deployment`

`kubectl get events`

`kubectl get pod,svc`

`kubectl get replicaset`

`kubectl edit deployment nginx-depl`

### debugging

`kubectl logs {pod-name}`

`kubectl exec -it {pod-name} -- bin/bash`

`kubectl describe service httpd-service`

### create or edit config file
###### test
`vim nginx-deployment.yaml`

`kubectl apply -f nginx-deployment.yaml`



  # Ressource im Cluster aktualisieren (auch für das Hinzufügen neuer Ressourcen)
  kubectl apply -f dashboard-admin-sa.yml
  # vorhandene Ressource wird durch neue Version ersetzt
  kubectl replace
  
  #  Ressourcen im Cluster anzeigen
  kubectl get all 
  kubectl get deployments
  kubectl get pods
  kubectl get events
  # Alternativ service/svc
  kubectl get services 
  kubectl get pod,svc

  # detaillierte Infos zu Ressource bekommen
  kubectl describe service httpd-service


  # Config des Clusters anzeigen
  kubectl config view

  # Ausgabe der URL eines gewählten Services
  minikube service mysql-service --url


  # Erstellen eines SErvices Typ LoadBalancer
  kubectl expose deployment hello-node --type=LoadBalancer --port=8080
  
  # Service URL wird automatisch im Browser geöffnet bztw. nur die URL ausgegeben
  minikube service hello-node
  minikube service hello-node --url
  
  # Addons Minikube verwalten
  minikube addons list (alle anzeigen)
  minikube addons disable metrics-server 
  minikube addons enable dashboard

  # Dashboard
  minicube dashboard 

  # INfos zum Cluster anzeigen
  kubectl cluster-info

  
  445  kubectl logs pod/mysql-deployment-54b5fdd84c-fzd27
  446  kubectl get svc mysql-service
  447  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
  448  kubectl apply -f dashboard-admin-sa.yaml
  449  kubectl apply -f dashboard-admin-sa.yml
  
  # Proxy, der den Zugriff auf Pods ohne Service zulässt
  kubectl proxy
  # Podname in externer Variable speichern und auf Pod zugreifen
  export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
  echo $POD_NAME
  curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/

  # Zusätzliche Befehle auf Pod ausführen (Umgebungsvariablen anzeigen, Shell öffnen)
  kubectl exec $POD_NAME -- env
  kubectl exec -it $POD_NAME -- bash