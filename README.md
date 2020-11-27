https://artifacthub.io/packages/helm/prometheus-community/prometheus


# Prereq
* kubectl and helm are in PATH.
* access to a local or remote Kubernetes cluster

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create ns monitoring
```

# prometheus
Create the original value files - which will be customized
```
helm show values prometheus-community/prometheus --version 12.0.1 > prometheus/values.yaml
```

(Optional) Review rendered templates
```
helm template prom prometheus-community/prometheus \
    --version 12.0.1 \
    --output-dir=out \
    -f prometheus/values.yaml
```

Install chart
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

# Sample applications
```
k create ns sample
```
## Java - Spring with Actuator
```
git clone https://github.com/serbangilvitu/wave.git
cd wave
helm install wave k8s/wave \
    -n sample \
    -f k8s/wave/values.yaml
```

http://localhost:9090/graph?g0.range_input=5m&g0.stacked=1&g0.expr=sum(jvm_memory_used_bytes%7Barea%3D%22heap%22%2Ckubernetes_name%3D%22wave%22%7D)&g0.tab=0