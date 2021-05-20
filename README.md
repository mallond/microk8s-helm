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
sudo microk8s enable dashboard dns registry helm helm3 ingress 
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

kubectl config view --raw > ~/.kube/config

```

## Charts
### Ingress
https://github.com/kubernetes/ingress-nginx/tree/master/charts/ingress-nginx
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install myingress ingress-nginx/ingress-nginx
helm show values ingress-nginx/ingress-nginx
kubectl get namespace
kubectl get pods -n ingress

# Lets add a couple of services
kubectl create -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/hello-kubernetes.yaml
kubectl create -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/service-apple.yaml
kubectl create -f https://raw.githubusercontent.com/mallond/microk8s-helm/main/service-banana.yaml

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
  Test
  curl https://10.152.183.22/ -k
  
  # Expose the ingress servcie
  kubectl get svc myingress-ingress-nginx-controller
  NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
  myingress-ingress-nginx-controller   LoadBalancer   10.152.183.22   <pending>     80:31640/TCP,443:31735/TCP   68m
  
  kubectl patch svc myingress-ingress-nginx-controller  -n default -p '{"spec": {"externalIPs":["172.31.112.14"]}}'

   
```


### InfluxDB
https://artifacthub.io/packages/helm/bitnami/influxdb
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install myinfluxdb bitnami/influxdb
helm show values bitnami/influxdb

# Add this to ingress
myinfluxdb 
kubectl edit  ingress example-ingress
-----------------
      - backend:
          service:
            name: myinfluxdb
            port:
              number: 8086
        path: /influxdb
        pathType: ImplementationSpecific
------------------ 

Test
curl https://10.152.183.22/influxdb -k


```

### Jenkins
https://www.jenkins.io/doc/book/installing/kubernetes/
```
kubectl get namespaces

```

