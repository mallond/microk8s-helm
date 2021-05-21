# microk8s-helm


![download](https://user-images.githubusercontent.com/993459/111545821-ea5d5880-8733-11eb-9352-d22f812e9fb0.png)

## Microk8s Base Install - Ubuntu 20.x
```
sudo snap install microk8s --classic --channel=1.21/stable
```
```
sudo microk8s status --wait-ready 
```
```
sudo microk8s enable dashboard dns registry helm3 prometheus 
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

## Helm Install and Setup
```
sudo snap install helm --classic

mkdir ~/.kube
kubectl config view --raw > ~/.kube/config

```

## Out of the Box with Microk8S
```
# List Monitoring Tools
microk8s kubectl get pods -n monitoring

# Prometheus UI
kubectl port-forward -n monitoring service/prometheus-k8s --address 0.0.0.0 9090:9090

# Grafana UI
kubectl port-forward -n monitoring service/grafana --address 0.0.0.0 3000:3000

```

## Charts


### InfluxDB - Port 8085
https://artifacthub.io/packages/helm/bitnami/influxdb
```
# Supporting Commands Used
helm repo list 
helm delete influxdb -n influxdb
helm list -a
kubectl describe svc influxdb -n influxdb
kubectl get pods -n influxdb
kubectl describe pods influxdb-767fb4fb57-lbrzp -n influxdb
kubectl get svc -n influxdb

# Install 
kubectl create namespace influxdb
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install influxdb --set auth.admin.username=admin,auth.admin.password=password,influxdb.service.type=LoadBalancer,influxdb.service.port=8085 -n influxdb bitnami/influxdb
kubectl patch svc influxdb -n influxdb -p '{"spec": {"externalIPs":["172.31.119.23"]}}'

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

### Jenkins - Port 8091
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

## Install
kubectl create namespace jenkins
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install jenkins --set jenkinsUser=admin,jenkinsPassword=password,service.port=8091 bitnami/jenkins -n jenkins
- Set up external address
kubectl patch svc jenkins -n jenkins -p '{"spec": {"externalIPs":["172.31.21.205"]}}'

  
```

## Ingress
https://artifacthub.io/packages/helm/bitnami/nginx-ingress-controller
```
## Install
kubectl create namespace ingress
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install my-release bitnami/nginx-ingress-controller

helm install ingress --set image.pullPolicy=Always bitnami/nginx-ingress-controller -n ingress
- Set up external address



```



# Backup and Recovery  
 https://howchoo.com/kubernetes/kubectl-cp-copy-files-to-from-pods
```
kubectl get pod influxdb-6bc5449b4-dqjbd
kubectl exec --stdin --tty influxdb-6bc5449b4-dqjbd -n influxdb -- /bin/bash 

kubectl get PersistentVolumeClaim -n influxdb
kubectl cp <local_path> <namespace>/<pod_name>:/tmp/backup/
kubectl cp influxdb-6bc5449b4-dqjbd:/etc/influxdb/influxdb.conf ~/influxdb_backups -n influxdb

kubectl cp my-pod:my-file my-file
/etc/influxdb/influxdb.conf

```

# Notes
```
sudo snap remove microk8s
```
