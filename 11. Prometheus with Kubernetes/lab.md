# Course Content:

1. Basic understanding of Kubernetes 
2. Helm
3. Install prometheus-community/kube-prometheus-stack
4. Inspect all the components one by one.
	a. Service
	b. configMaps
	c. Secrets
5. How to open the GUI and demonstrate the tabs
	a. Prometheus 
	b. AlertManager
	c. PushGateway
6. Install MongoDB Exporter
# 


### How to install the GIT and then enable Helm?

```
mkdir /data ; cd /data
yum install -y git
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh 
./get_helm.sh 
```
### How to add the GIT binary into our local Path?

```
vi /root/.bashrc
```
```
PATH="$HOME/.local/bin:$HOME/bin:/usr/local/bin:$PATH"
```

### How to enable auto complete command?
```
helm completion bash > /etc/bash_completion.d/helm
```
### logout and login again.

### How to list the helm repo?
```
helm repo list 
```

```
https://github.com/prometheus-community/helm-charts/
```
### How to add the prometheus-community repo in helm?
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```
helm repo list 
```
### How to update the repo?
```
helm repo update
```
```
helm show values prometheus-community/kube-prometheus-stack > values.yaml
```
### Install the kube-prometheus-stack under prometheus-monitoring namespace?
```
helm install prometheus prometheus-community/kube-prometheus-stack --create-namespace --namespace prometheus-monitoring
```
### It will add all objects and adding the labels "release=prometheus"
```
kubectl --namespace prometheus-monitoring get pods -l "release=prometheus"
```
### Other way to list all the pods inside this namespace.
```
kubectl -n prometheus-monitoring get pods
```

```
kubectl -n prometheus-monitoring get all
```

```
kubectl -n prometheus-monitoring get svc
```

```
curl http://10.102.113.206:9090
```

```
curl http://$(kubectl -n prometheus-monitoring get service/prometheus-kube-prometheus-prometheus | awk '{print $3}' | grep -v CLUSTER):9090
```
### Expose the service so that we can open the Prometheus GUI from our local browser.
```
kubectl -n prometheus-monitoring expose service prometheus-kube-prometheus-prometheus --type=NodePort --target-port=9090 --name=prometheus-server-ext 
```

```
[root@master1 data]# kubectl -n prometheus-monitoring get svc | grep -i nodeport
alertmanager-server-ext                   NodePort    10.110.231.118   <none>        9093:31902/TCP,9094:32499/TCP,9094:32499/UDP   84d
prometheus-server-ext                     NodePort    10.111.173.174   <none>        9090:30430/TCP,8080:31161/TCP                  84d
```

### Open the GUI 
### For Alertmanager: 192.168.1.31:9094
### For Promethues : 192.168.1.31:30430


```
kubectl -n prometheus-monitoring expose service alertmanager-operated --type=NodePort --target-port=9093 --name=alertmanager-server-ext
```

### List all crd (custom resource defination).
```
kubectl -n prometheus-monitoring get crd
```
### List all ConfigMaps (cm)
```
kubectl -n prometheus-monitoring get cm 
```

```
kubectl -n prometheus-monitoring describe cm prometheus-prometheus-kube-prometheus-prometheus-rulefiles-0
```


### How to add new custom rules?
```
vi /data/values.yaml
```
### Search for "additionalPrometheusRulesMap" and add below lines. 
```
additionalPrometheusRulesMap:
  custom-rules:
    groups:
    - name: example.test-anish
      rules:
      - alert: ServerDownupstream 1
        expr: 1
        annotations:
          summary: Server Upstream1 Down
        labels:
          severity: critical
          service: app
          env: dev
```
### This time, we will use "upgrade" sub-command with helm.
```
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -f values.yaml --namespace prometheus-monitoring
```


### Check the pods:
```
kubectl -n prometheus-monitoring get pods
```
### Post checks, check the Prometheus GUI and AlertManager GUI.

### How to list servicemonitors (CRD)
```
kubectl -n prometheus-monitoring get servicemonitors.monitoring.coreos.com 
```
### Once can observe that there are two types of CRDs (Custom Resource Definition).
```
kubectl -n prometheus-monitoring get crd | grep -v coreos.com
```
I am using calico for networking. This Prometheus stack, identify it and create these networkpolicies for us. 
For an example, the globalnetworkpolicies.crd.projectcalico.org is a Custom Resource Definition (CRD) used by Project Calico, 
a popular open-source networking and network security solution for containers, virtual machines, and native host-based workloads. 
This CRD defines the schema for Global Network Policies in Calico.

```
kubectl -n prometheus-monitoring get crd | grep -i coreos.com
```
```html
CoreOS was a company and a set of technologies focused on improving the security, reliability, and scalability of internet
infrastructure through containerization and orchestration.

Key Technologies and Contributions:
----------------------------------
CoreOS Container Linux: A lightweight, container-optimized operating system designed to run containers securely and at scale.
It featured automatic updates and minimal overhead, making it ideal for large-scale deployments.

etcd: A distributed key-value store that provides a reliable way to store data across a cluster of machines.

Flannel: A virtual network fabric for containers, designed to give each container its own IP address.

Tectonic: An enterprise Kubernetes platform that included additional features for security, monitoring, and management.

Quay: A private container registry that allows users to store, build, and deploy container images securely. It includes
features like image scanning and repository mirroring.
```
The prometheuses.monitoring.coreos.com is a Custom Resource Definition (CRD) used by the Prometheus Operator, which is a 
tool developed by CoreOS (now part of Red Hat) to simplify the deployment and management of Prometheus instances on Kubernetes.


```
kubectl -n prometheus-monitoring describe ServiceMonitor prometheuses.monitoring.coreos.com
```

### How to restart the NodeExporter service?


```
kubectl -n prometheus-monitoring rollout restart ds prometheus-prometheus-node-exporter 
```















### How to enable the MongoDB exporter?
### We can also check the default values what it comes if we install the mmongdb exporter.

```
helm show values prometheus-community/prometheus-mongodb-exporter > mongo_value.yaml
```


### Add these values as we will going to install mongodb exporter on namespace "prometheus-monitoring" and add the release label, it is important.
```
cat <<EOF>> mongo_value.yaml 
namespace: prometheus-monitoring
prometheus-mongodb-exporter:
  enabled: true
  mongodb:
    uri: "mongodb://admin:password@localhost:27017/mydatabase"

prometheus:
  serviceMonitor:
    enabled: true
    namespace: prometheus-monitoring
    additionalLabels:
      release: prometheus
EOF
```
### Let's install the mongodb expoter with these values.
```
helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f mongo_value.yaml  -n prometheus-monitoring 
```
###  Check the mongodb pods, services, deployment
```
kubectl -n prometheus-monitoring get deployments.apps,svc,pods | grep mongo 
```

```
kubectl -n prometheus-monitoring get svc
```

### Expose the monogdb service, like we did for prometheus and alertManager.
```
kubectl -n prometheus-monitoring expose service mongodb-exporter-prometheus-mongodb-exporter --type=NodePort --target-port=9216 --name=mongodb-exporter-ext
```
### Check the dynamci port number.
```
[root@master1 data]# kubectl -n prometheus-monitoring get svc mongodb-exporter-ext exposed
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mongodb-exporter-ext   NodePort   10.99.134.172   <none>        9216:31325/TCP   12s
```

### Open the Browser and check the MongoDB URI.
```
http://192.168.1.31:31325/metrics
```
### As of now, we can access the newly installed MongoDB exporter but this service is not showing in our Prometheus GUI.
### Let's create serviceMonitor crd . With the help of this, prometheus will take into account.
```
kubectl -n prometheus-monitoring get servicemonitors.monitoring.coreos.com
```
```
cat <<EOF | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
    meta.helm.sh/release-name: mongodb-exporter
    meta.helm.sh/release-namespace: prometheus-monitoring
  labels:
    app.kubernetes.io/instance: mongodb-exporter
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: prometheus-mongodb-exporter
    app.kubernetes.io/version: 0.40.0
    helm.sh/chart: prometheus-mongodb-exporter-3.6.0
    release: prometheus
  name: mongodb-exporter-prometheus-mongodb-exporter
  namespace: prometheus-monitoring
spec:
  endpoints:
  - interval: 30s
    port: metrics
    scrapeTimeout: 10s
  namespaceSelector:            # Select the namespace
    matchNames:
    - prometheus-monitoring     # It will select 
  selector:
    matchLabels:
      app.kubernetes.io/instance: mongodb-exporter
      app.kubernetes.io/name: prometheus-mongodb-exporter
EOF
```

```
kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```
### Check  the Prometheus GUI 


### Search for "prometheus-kube-prometheus-kube" in the target option. You will observe that some services are not working or status is down.

![image](https://github.com/user-attachments/assets/b459c9f4-0269-48be-8047-ecd37cd617a2)

##
### How to resolve these problems. For kube-controller-manager
### We know that this pod is created statically. We need to updat the static yaml file and change the IP "127.0.0.1" to "0.0.0.0".
```
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```
```
kubectl -n kube-system get pods/kube-controller-manager-master1.example.com -o yaml
```
## Post check, Let's check the Prometheus GUI.

### Same treatment for kube-scheduler.
```
vi /etc/kubernetes/manifests/kube-scheduler.yaml
```
```
kubectl -n kube-system get pods/kube-scheduler-master1.example.com -o yaml
```
## Post check, Let's check the Prometheus GUI.
```
var=`kubectl -n prometheus-monitoring get svc prometheus-server-ext| awk -F ":"  '/9090/ {print $2}' | cut -d "/" -f1`; echo "http://192.168.1.31:$var"
```

### For kube-proxy, there is no satic pod. Actually, it is being managed by daemmonset and all the configuration are stored on ConfigMap. Thus, we have to modify the configMap and then restart the daemonSet.
```
kubectl -n kube-system get cm
```
```
kubectl -n kube-system edit configmaps kube-proxy
```
```
look for "metricsBindAddress"
metricsBindAddress: 0.0.0.0:10249
```
```
kubectl -n kube-system get all
```
```
kubectl -n kube-system rollout restart daemonset.apps/kube-proxy
```
## Post check, Let's check the Prometheus GUI.
```
var=`kubectl -n prometheus-monitoring get svc prometheus-server-ext| awk -F ":"  '/9090/ {print $2}' | cut -d "/" -f1`; echo "http://192.168.1.31:$var"
```

### For etcd, we just need to change the IP "127.0.0.1" to 0.0.0.0 for only "--listen-metrics-urls"
```
[root@master1 data]# cat /etc/kubernetes/manifests/etcd.yaml | grep listen-metrics-urls
    - --listen-metrics-urls=http://127.0.0.1:2381

[root@master1 data]# cat /etc/kubernetes/manifests/etcd.yaml | grep listen-metrics-urls
    - --listen-metrics-urls=http://0.0.0.0:2381
```
```
vi /etc/kubernetes/manifests/etcd.yaml
```
### After updating the etcd pod yaml file, we have to wait.
```
kubectl get nodes
```
## Post check, Let's check the Prometheus GUI.
```
var=`kubectl -n prometheus-monitoring get svc prometheus-server-ext| awk -F ":"  '/9090/ {print $2}' | cut -d "/" -f1`; echo "http://192.168.1.31:$var"
```

## How to uninstall the MongoDb exporter?
```
helm uninstall mongodb-exporter -n prometheus-monitoring
```
### Post checks, we can check the pods.
```
kubectl -n prometheus-monitoring get pods | grep mongo
```

### Need to delete the servicemonitor. First, identify the servicemonitor name.
```
kubectl -n prometheus-monitoring get servicemonitors.monitoring.coreos.com 
```

```
kubectl -n prometheus-monitoring delete servicemonitors.monitoring.coreos.com mongodb-exporter-prometheus-mongodb-exporter
```

### Also, need to delete the service. Identify the service name. 
```
kubectl -n prometheus-monitoring get service
```

```
kubectl -n prometheus-monitoring delete service mongodb-exporter-ext
```
## Remove the value.yaml file that we created when we install the mongodb.
```
rm -rf  /data/mongo_value.yaml
```
### How to uninstall the Prometheus from stack?
```
helm uninstall prometheus -n prometheus-monitoring
```

```
rm -rf /data/values.yaml
```

### You will observe that all the servicemonitors deleted. Because it was created by Helm. 
```
kubectl -n prometheus-monitoring get servicemonitors.monitoring.coreos.com 
```

### Check the services
```
kubectl -n prometheus-monitoring get service
```

### Let's delete these remaining services.
```
kubectl -n prometheus-monitoring delete service/alertmanager-server-ext service/prometheus-server-ext
```


### How to delete the namespace?
```
kubectl delete namespaces prometheus-monitoring 
```

### How to remove the helm repo ?

```
/usr/local/bin/helm repo list 
```
```
/usr/local/bin/helm repo remove prometheus-community
```
```
rm -rf /data/get_helm.sh


