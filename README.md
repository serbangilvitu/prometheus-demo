
# prometheus-demo
Demo - set up Prometheus and Prometheus Blackbox exporter, a sample service, and send a message on Slack when the service becomes unavailable.

## Prerequsites
* kubectl and helm are in PATH.
* access to a local or remote Kubernetes cluster
* a Slack account with a configured webhook

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create ns monitoring
```

## prometheus
### (Optional) Review value file changes
You can see the value file changes by comparing `prometheus/values.yaml`with the original value file
```
helm show values prometheus-community/prometheus --version 12.0.1 > prometheus/original-values.yaml
```

### Update Slack webhook in the value file
Replace `<slack-webhook>` with the actual webhook.


### (Optional) Review rendered templates
```
helm template prom prometheus-community/prometheus \
    --version 12.0.1 \
    --output-dir=out \
    -n monitoring \
    -f prometheus/values.yaml
```

### Deployment
```
helm install prom prometheus-community/prometheus \
    --version 12.0.1 \
    -n monitoring \
    -f prometheus/values.yaml
```

(Optional) List releases in the monitoring namespace
```
helm ls -n monitoring
```

(Optional) Use port forwarding to access the services
```
kubectl port-forward -n monitoring svc/prom-kube-state-metrics 8080:8080
kubectl port-forward -n monitoring svc/prom-prometheus-server  9090:80
```

## prometheus-blackbox-exporter
### (Optional) Review value file changes
You can see the value file changes by comparing `prometheus-blackbox-exporter/values.yaml`with the original value file
```
helm show values prometheus-community/prometheus-blackbox-exporter --version 4.10.1 > prometheus-blackbox-exporter/original-values.yaml
```

### (Optional) Review rendered templates
```
helm template pbe prometheus-community/prometheus-blackbox-exporter \
    --version 4.10.1 \
    --output-dir=out \
    -n monitoring \
    -f prometheus-blackbox-exporter/values.yaml
```

### Deployment
```
helm install pbe prometheus-community/prometheus-blackbox-exporter \
    --version 4.10.1 \
    -n monitoring \
    -f prometheus-blackbox-exporter/values.yaml
```

## Sample applications
Create a new namespace
```
kubectl create ns sample
```

Some explanations regarding the annotations used for Kubernetes services:
* `prometheus.io/probe: "true"` Blackbox exporter will only probe the services which have this annotation
* if the application is exposing metrics (e.g. via Actuator), then the following annotations are also required
```
prometheus.io/scrape: "true"
prometheus.io/port: "{{ .Values.service.port }}"
prometheus.io/path: "/actuator/prometheus"
```

### Java - Spring with Actuator
Uses [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html)
[Code](https://github.com/serbangilvitu/wave)
```
pushd sample-apps/wave
helm install wave . \
    -n sample \
    -f values.yaml
popd
```

# Probe status
http://localhost:9090/graph?g0.range_input=1h&g0.expr=probe_success%7Bkubernetes_name%3D%22wave%22%2Ckubernetes_namespace%3D%22sample%22%7D&g0.tab=0

# Alerts
Test alerts by scaling down the replicas
```
k scale -n sample deployment wave --replicas=0
```

To receive a resolved notification, scale back to 1
```
k scale -n sample deployment wave --replicas=1
```