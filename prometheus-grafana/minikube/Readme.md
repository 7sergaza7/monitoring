
# Running postgresql on minikube with Prometheus-Grafana

## Minikube minimum setup on Windows
```
minikube start
minikube addons enable metrics-server
minikube addons enable auto-pause
minikube addons enable ingress
```

## Prometheus helm install
```
helm repo add prometheus-community <https://prometheus-community.github.io/helm-charts>
helm repo update

helm install prometheus prometheus-community/prometheus --version 27.16.0 --namespace=monitoring
expose service
```
kubectl expose service prometheus-server --type=NodePort 
--target-port=9090 --name=prometheus-server-ext
```
Need to expose it in cli due to docker for windows (no need for hyperv)
```
minikube service prometheus-server-ext -n monitoring
```
## Grafana helm install

```
helm repo add grafana https://grafana.github.io/helm-charts --namespace grafana
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" 
echo "<sercret>" | base64 --decode
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext -n grafana
```

! need to expose it in cli due to docker for windows (no need for hyperv)

```
minikube service grafana-ext -n grafana
```

## Addint datasources and dashboards on grafana

- Add datasource of external prometheus server (nodeport of prometheus-server-ext).
  i.e.: <http://192.168.49.2:32619>
  
- Kubernetes default dashboard id to import: [18283](https://grafana.com/grafana/dashboards/18283-kubernetes-dashboard/)

- Postgresql dashboard: [9628](https://grafana.com/grafana/dashboards/9628-postgresql-database/)



<!-- # prometheus grafana

1. `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`

1. `helm repo update`

1. `helm install prometheus prometheus-community/kube-prometheus-stack --namespace=prometheus --create-namespace --wait`

## minikube map port forward

  Start-Process kubectl  -ArgumentList "port-forward deployment/prometheus-grafana 3000 --namespace=prometheus"  -NoNewWindow
  Start-Process kubectl  -ArgumentList "port-forward service/prometheus-prometheus-node-exporter 9100   --namespace=prometheus"  -NoNewWindow
  Start-Process kubectl  -ArgumentList "port-forward service/prometheus-operated  9090 --namespace=prometheus"  -NoNewWindow

# postgresql

- `helm repo add bitnami https://charts.bitnami.com/bitnami`

- `helm install postgres bitnami/postgresql --values .\postress-values.yaml --namespace db --create-namespace`

1. export POSTGRES_PASSWORD=$(kubectl get secret --namespace db postgres-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

1. kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace db --image docker.io/bitnami/postgresql:17.4.0-debian-12-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD" --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432 -->