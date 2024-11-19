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
yum install -y git
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh 
./get_helm.sh 
```
### How to add the GIT binary into our local Path?

```
[root@master1 ~]# pwd
/root
vi .bashrc
```

```
vi .bashrc
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


### How to modify cm (configMap)?

```
kubectl -n prometheus-monitoring edit cm prometheus-prometheus-kube-prometheus-prometheus-rulefiles-0
```

### Check the pods:
```
kubectl -n prometheus-monitoring get pods
```
```
kubectl -n prometheus-monitoring get statefulsets.apps 
```
### If we do any modification, then we need to restart the prometheus.
```
kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```

```
kubectl -n prometheus-monitoring get pods
```
### How to list servicemonitors (CRD)
```
kubectl -n prometheus-monitoring get servicemonitors.monitoring.coreos.com 
```

```
kubectl -n prometheus-monitoring describe ServiceMonitor prometheus-kube-prometheus-operator
```

### How to restart the NodeManager service?


```
kubectl -n prometheus-monitoring rollout restart ds prometheus-prometheus-node-exporter -n prometheus-monitoring
```















### How to enable the MongoDB exporter?
### We can also check the default values what it comes if we install the mmongdb exporter.

```
helm show values prometheus-community/prometheus-mongodb-exporter > mongo_value.yaml
```

### Need to remove.
```
cat <<EOF> mongo_value.yaml 
mongodb:
  uri: "mongodb://mongodb-service-app:27017"


serviceMonitor:
  additionalLabels:
    release: Prometheus
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
helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f mongo_value.yaml  --n prometheus-monitoring 
```
###  Check the mongodb pods, services, deployment
```
kubectl -n prometheus-monitoring get deployments.apps,svc,pods | grep mongo 
```

### Expose the monogdb service, like we did for prometheus and alertManager.
```
kubectl -n prometheus-monitoring expose service mongodb-exporter-prometheus-mongodb-exporter --type=NodePort --target-port=9216 --name=mongodb-exporter-ext
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


### Just for your information 

```
kubectl -n prometheus-monitoring get Prometheus prometheus-kube-prometheus-prometheus -o yaml
```

```
kubectl get pods -n kube-system --show-labels | grep controller
```

```
[root@master1 ~]# kubectl get pods -n kube-system --show-labels | grep controller
calico-kube-controllers-658d97c59c-qb5vq      1/1     Running   36 (12h ago)   93d     k8s-app=calico-kube-controllers,pod-template-hash=658d97c59c
calico-node-58nw8                             1/1     Running   32 (12h ago)   93d     controller-revision-hash=574c44bccd,k8s-app=calico-node,pod-template-generation=1
```

```
kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-controller-manager
```

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
    meta.helm.sh/release-name: prometheus
    meta.helm.sh/release-namespace: prometheus-monitoring
  creationTimestamp: "2024-11-10T13:34:33Z"
  generation: 2
  labels:
    app: kube-prometheus-stack-kube-controller-manager
    app.kubernetes.io/instance: prometheus
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/part-of: kube-prometheus-stack
    app.kubernetes.io/version: 66.1.0
    chart: kube-prometheus-stack-66.1.0
    heritage: Helm
    release: prometheus
  name: prometheus-kube-prometheus-kube-controller-manager
  namespace: prometheus-monitoring
  resourceVersion: "1010213"
  uid: ae605ff7-820b-466c-9318-7c72d24dfbcf
spec:
  endpoints:
  - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    port: http-metrics
    scheme: https
    tlsConfig:
      caFile: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      insecureSkipVerify: true
  jobLabel: jobLabel
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      app: kube-prometheus-stack-kube-controller-manager
      k8s-app: calico-kube-controllers
      release: prometheus
```

```
kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```

```
http://192.168.1.31:30144/targets?search=
```




```
kubectl get pods -n kube-system --show-labels | grep etcd
```

```
kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-etcd
```

```
kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```

```
kubectl get pods -n kube-system --show-labels | grep proxy
```

```
[root@master1 ~]# kubectl get pods -n kube-system --show-labels | grep etcd
etcd-master1.example.com                      1/1     Running   37 (12h ago)   93d     component=etcd,tier=control-plane
[root@master1 ~]# kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-etcd
servicemonitor.monitoring.coreos.com/prometheus-kube-prometheus-kube-etcd edited
[root@master1 ~]#  kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
statefulset.apps/prometheus-prometheus-kube-prometheus-prometheus restarted
[root@master1 ~]# 
[root@master1 ~]# 
[root@master1 ~]# kubectl get pods -n kube-system --show-labels | grep proxy
kube-proxy-fmfkc                              1/1     Running   36 (12h ago)   93d     controller-revision-hash=744b5455d5,k8s-app=kube-proxy,pod-template-generation=1
kube-proxy-gq66f                              1/1     Running   31 (12h ago)   93d     controller-revision-hash=744b5455d5,k8s-app=kube-proxy,pod-template-generation=1
kube-proxy-m8p7b                              1/1     Running   36 (12h ago)   93d     controller-revision-hash=744b5455d5,k8s-app=kube-proxy,pod-template-generation=1
```

```
kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-proxy
```
```
kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```
```
kubectl get pods -n kube-system --show-labels | grep scheduler
```


```
[root@master1 ~]# kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-proxy
servicemonitor.monitoring.coreos.com/prometheus-kube-prometheus-kube-proxy edited
[root@master1 ~]# kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus

[root@master1 ~]# kubectl get pods -n kube-system --show-labels | grep scheduler
kube-scheduler-master1.example.com            1/1     Running   130 (8h ago)   93d     component=kube-scheduler,tier=control-plane
```

```
kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-scheduler
```

```
kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```
```
kubectl get pods -n kube-system --show-labels | grep scheduler
```


```
[root@master1 ~]# kubectl -n prometheus-monitoring edit servicemonitors.monitoring.coreos.com prometheus-kube-prometheus-kube-scheduler
servicemonitor.monitoring.coreos.com/prometheus-kube-prometheus-kube-scheduler edited
[root@master1 ~]# kubectl -n prometheus-monitoring rollout restart statefulset prometheus-prometheus-kube-prometheus-prometheus
```

## How to uninstall the MongoDb exporter?
```
helm uninstall mongodb-exporter -n prometheus-monitoring
```


```
kubectl -n prometheus-monitoring get pods | grep mongo
```

### Need to delete the service also. First, identify the servicemonitor name.
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


### How to uninstall the Prometheus from stack?
```
helm uninstall prometheus -n prometheus-monitoring
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



