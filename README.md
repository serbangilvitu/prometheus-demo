https://artifacthub.io/packages/helm/prometheus-community/prometheus

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

# Prereq
```
kubectl create ns monitoring
```

# kube-state-metrics
Create the original value files - which will be customized
```
helm show values bitnami/kube-state-metrics --version 1.1.0  > kube-state-metrics/values.yaml
```

(Optional) Review rendered templates
```
helm template kube-state-metrics bitnami/kube-state-metrics \
    --version 1.1.0 \
    --output-dir=out \
    -f kube-state-metrics/values.yaml
```

Install chart
```
helm install kube-state-metrics bitnami/kube-state-metrics \
    --version 1.1.0 \
    -n monitoring \
    -f kube-state-metrics/values.yaml
```

(Optional) List releases in the monitoring namespace
```
helm ls -n monitoring
```

(Optional) Use port forwarding to access the metrics
```
kubectl port-forward --namespace monitoring svc/kube-state-metrics-mon 9100:8080
```

# prometheus
Create the original value files - which will be customized
```
helm show values bitnami/kube-prometheus --version 3.1.1 > prometheus/values.yaml
```
