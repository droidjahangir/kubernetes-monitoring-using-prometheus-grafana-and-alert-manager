### Run minikube
```
minikube start --driver=docker
```

* run pods using helmfile `helmfile sync`
* destroy pods `helmfile destroy`

**helm repo for prometheus chat options**
https://github.com/prometheus-community/helm-charts/


add prometheus repo
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

### Deploy prometheus operator stack
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
helm ls
```

`kubectl --namespace monitoring get pods -l "release=monitoring"` ---> get all pods with specific namespace and label

`helm ls --namespace monitoring` ---> print list the releases that have been deployed using Helm

**get all information about prometheus**
```
kubectl get all -n monitoring
```

**access prometheus UI using port-forwarding**
```
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring &
```
access from this link ---> `127.0.0.1:9090`

**access grafana**
```
kubectl port-forward svc/monitoring-grafana 8080:80 -n monitoring &
```
access from this link ---> `127.0.0.1:8080`

`user: admin pwd: prom-operator` ---> username and password for grafana user interface login

**access alert manager UI using port forwarding**
```
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093 &
```
access from this link ---> `127.0.0.1:9093`


**Trigger CPU spike with many requests**

Deploy a busybox pod so we can curl our application so that we can make cpu stress
```
kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm
```
make a file inside this pod as `touch test.sh` and make it executable from this command `chmod +x test.sh`
```bash
for i in $(seq 1 10000)
do
  curl http://192.168.49.2:30054/ > test.txt
done

```
then run this command to make 10000 request to out application endpoint

**this url should not be same but we can create this url by getting this minikube cluster ip and frontend service nodeport address by applying those command**
```
minikube ip
kubectl get all
```

## Alert manager

we can build query functions from prometheus documentation
https://prometheus.io/docs/prometheus/latest/querying/functions/

**Kubernetes alert runbook**
https://github.com/pingcap/monitoring/blob/master/k8s-cluster-monitor/vendor/kubernetes-mixin/runbook.md

**build custom monitoring rule for spec section**
https://docs.openshift.com/container-platform/4.10/rest_api/monitoring_apis/prometheus-monitoring-coreos-com-v1.html

## Get alert when cpu used over 50%

create alert-rules.yaml file to create rule. Then apply this rule
create alert rules by applying this file in kubectl
```
kubectl apply -f alert-rules.yaml
```

get prometheus rule info using this command
```
kubectl get PrometheusRule -n monitoring
```

get details info about this rule
```
kubectl logs prometheus-monitoring-kube-prometheus-prometheus-0 -n monitoring -c config-reloader
```

**Create CPU stress**
```
kubectl delete pod cpu-test; kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 60s --metrics-brief
```

## Create alert manager

link for this alert manager config 
https://docs.openshift.com/container-platform/4.10/rest_api/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1alpha1.html

apply alert-manager-configuration.yaml file to implement.


## Monitor third-party app like redis

redis exporter 
https://github.com/oliver006/redis_exporter

install helmchart for redis-exporter
https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-redis-exporter

**Deploy redis exporter**
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml
```

then create `redis-rules.yaml` and `redis-values.yaml` files and apply those files using kubectl

awesome prometheus alert rules for configuring different alert rule
https://samber.github.io/awesome-prometheus-alerts/

we can use ready made alert rule for different tools from this awesome prometheus alert rules.


we can integrate grafana dashboard with redis using this tool
https://grafana.com/grafana/dashboards/14091-redis-dashboard-for-prometheus-redis-exporter-1-x/

we just need dashboard id to import grafana dashboard into grafana.


## Monitor nodejs app
monitor nodejs app using `prom-client` npm package

