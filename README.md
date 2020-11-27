```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Create the original value files - which will be customized
```
helm show values bitnami/kube-prometheus --version 3.1.1 > values.yaml
```