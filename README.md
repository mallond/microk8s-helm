# microk8s-helm


![download](https://user-images.githubusercontent.com/993459/111545821-ea5d5880-8733-11eb-9352-d22f812e9fb0.png)

## Base Install - Ubuntu 20.x
```
sudo snap install microk8s --classic --channel=1.21/stable
```
```
sudo microk8s status --wait-ready 
```
```
sudo microk8s enable dashboard dns registry helm3 ingress 
```
```
sudo snap alias microk8s.kubectl kubectl    
```
1. After the above install, you will get the message. re-login to have the alias enabled
```
kubectl
Insufficient permissions to access MicroK8s.
You can either try again with sudo or add the user cloud_user to the 'microk8s' group:

    sudo usermod -a -G microk8s cloud_user
    sudo chown -f -R cloud_user ~/.kube

The new group will be available on the user's next login.
```

## Helm
```
sudo snap install helm --classic

mkdir ~/.kube
kubectl config view --raw > ~/.kube/config

```

## Charts
### Ingress
https://artifacthub.io/packages/helm/bitnami/nginx-ingress-controller
```
# Supporting Commands Used
helm repo list 
helm list -a
helm delete ingress -n ingress
kubectl get pods -n ingress
kubectl describe pods ingress-767fb4fb57-lbrzp -n ingress
kubectl get svc -n ingress

# Install
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install ingress bitnami/nginx-ingress-controller -n ingress
kubectl patch svc ingress-nginx-ingress-controller  -n ingress -p '{"spec": {"externalIPs":["172.31.35.18"]}}'



# Lets add a couple of services

kubectl create -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/service-apple.yaml
kubectl create -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/service-banana.yaml
kubectl create -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/hello-kubernetes.yaml

# Apply the ingress service
kubectl apply -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/services-ingress.yaml
  Lets do a few tests
  kubectl get svc myingress-ingress-nginx-controller
  NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
  myingress-ingress-nginx-controller   LoadBalancer   10.152.183.22   <pending>     80:31640/TCP,443:31735/TCP   35m
  curl https://10.152.183.22/apple -k
  curl https://10.152.183.22/banana -k

# Modify the ingress service
kubectl describe ingress example-ingress
kubectl edit  ingress example-ingress
  Lets modify and add hello-kubernets service
   ---------------------
        - backend:
          service:
            name: hello-kubernetes
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
  -------------------------
```


### InfluxDB
https://artifacthub.io/packages/helm/bitnami/influxdb
```
# Supporting Commands Used
helm repo list 
helm delete influxdb -n influxdb
helm list -a
kubectl describe svc influxdb -n influxdb
kubectl get pods -n influxdb
kubectl describe pods influxdb-767fb4fb57-lbrzp -n influxdb
sudo ufw status verbose
sudo ufw allow 8089/tcp
kubectl get svc -n influxdb

# Install
kubectl create namespace influxdb
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install influxdb --set auth.admin.username=admin,auth.admin.password=password,ingress.enabled=true,influxdb.service.type=NodePort,influxdb.service.port=8086 -n influxdb bitnami/influxdb

# Add to Ingress
kubectl edit  ingress example-ingress
---------------------
        - backend:
          service:
            name: influxdb
            port:
              number: 8086
        path: /influxdb
        pathType: ImplementationSpecific
  -------------------------

kubectl patch svc influxdb -n influxdb -p '{"spec": {"externalIPs":["172.31.27.73"]}}'
```

### Jenkins
https://artifacthub.io/packages/helm/bitnami/jenkins
```
# Supporting Commands Used
helm repo list 
helm list -a
helm delete jenkins -n jenkins
kubectl get pods -n jenkins
kubectl describe pods jenkins-767fb4fb57-lbrzp -n jenkins
sudo ufw status verbose
sudo ufw allow 8081/tcp
kubectl scale deployment jenkins --replicas=0

## Ingress Jenkins Test
kubectl apply -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/jenkins-ingress.yaml
 

# Install
kubectl create namespace jenkins
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install jenkins --set jenkinsUser=admin,jenkinsPassword=password -n jenkins bitnami/jenkins 
kubectl get svc -n jenkins

# Add to Ingress
kubectl edit  ingress example-ingress
---------------------
        - backend:
          service:
            name: jenkins
            port:
              number: 80
        path: /jenkins
        pathType: ImplementationSpecific
  -------------------------
  
```

# Notes
```
sudo snap remove microk8s
```
