
# Running PostgreSQL on Minikube with Prometheus-Grafana

## Minikube Minimum Setup on Windows
```bash
minikube start
minikube addons enable metrics-server
minikube addons enable auto-pause
minikube addons enable ingress
```

## Prometheus Installation
```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/prometheus --version 27.16.0 --namespace=monitoring --create-namespace
```

### Expose Prometheus Service
```bash
# Expose Prometheus as NodePort service
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext --namespace=monitoring

# Access Prometheus (required for Docker for Windows)
minikube service prometheus-server-ext --namespace=monitoring
```

## Grafana Installation
```bash
# Add Grafana Helm repository
helm repo add grafana https://grafana.github.io/helm-charts --namespace grafana
helm repo update

# Get Grafana admin password
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Expose Grafana service
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext --namespace=grafana

# Access Grafana (required for Docker for Windows)
minikube service grafana-ext --namespace=grafana
```

## Adding Datasources and Dashboards to Grafana

1. **Add Prometheus Data Source**:
   - Configure external Prometheus server using the NodePort address of `prometheus-server-ext`
   - Example URL: `http://192.168.49.2:32619`

2. **Import Recommended Dashboards**:
   - Kubernetes dashboard: [18283](https://grafana.com/grafana/dashboards/18283-kubernetes-dashboard/)
   - PostgreSQL dashboard: [9628](https://grafana.com/grafana/dashboards/9628-postgresql-database/)

## Alternative Setup (Kube Prometheus Stack)

<details>
<summary>Click to expand alternative setup instructions</summary>

### Prometheus-Grafana Stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace=prometheus --create-namespace --wait
```

### Port Forwarding (Windows PowerShell)
```powershell
Start-Process kubectl -ArgumentList "port-forward deployment/prometheus-grafana 3000 --namespace=prometheus" -NoNewWindow
Start-Process kubectl -ArgumentList "port-forward service/prometheus-prometheus-node-exporter 9100 --namespace=prometheus" -NoNewWindow
Start-Process kubectl -ArgumentList "port-forward service/prometheus-operated 9090 --namespace=prometheus" -NoNewWindow
```

### PostgreSQL Installation
```bash
# Add Bitnami charts repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install PostgreSQL with values file
helm install postgres bitnami/postgresql --values ./postgres-values.yaml --namespace db --create-namespace

# Get PostgreSQL password
export POSTGRES_PASSWORD=$(kubectl get secret --namespace db postgres-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

# Run PostgreSQL client to connect to the database
kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace db \
  --image docker.io/bitnami/postgresql:17.4.0-debian-12-r15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432
```
</details>