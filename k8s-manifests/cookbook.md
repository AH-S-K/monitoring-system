# Create the dedicated namespace for monitoring components
kubectl create namespace monitoring-system

# Set the kubeconfig path for k3s
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# Add and update the VictoriaMetrics Helm repository
helm repo add vm https://victoriametrics.github.io/helm-charts/
helm repo update

# Install the VictoriaMetrics Operator with minimal resource footprint
helm upgrade --install vm-operator vm/victoria-metrics-operator \
  --namespace monitoring-system \
  --create-namespace \
  --qps 1 \
  --burst-limit 3 \
  --set operator.resources.requests.cpu=10m \
  --set operator.resources.requests.memory=40Mi \
  --set operator.resources.limits.cpu=250m \
  --set operator.resources.limits.memory=128Mi

# Apply secrets required by Grafana and VMAuth
kubectl apply -f 0.secrets.yaml

# Deploy the core VictoriaMetrics single-node database
kubectl apply -f 1-vmsingle.yaml

# Deploy VMAgent to scrape and remote-write metrics
kubectl apply -f 2-vmagent.yaml

# Deploy Node Exporter DaemonSet for host-level metrics
kubectl apply -f 3-node-exporter.yaml

# Configure VMAgent to scrape the Django backend metrics
kubectl apply -f 4-django-vmscrape.yaml

# Deploy VMAuth gateway and configure the admin user routing
kubectl apply -f 5-vmauth-security.yaml

# Deploy kube-state-metrics for Kubernetes cluster state metrics
kubectl apply -f 6-kube-state-metrics.yaml

# Deploy Grafana with persistent storage, sub-path configuration, and datasource provisioning
kubectl apply -f 7-grafana.yaml

# Apply Ingress rules to expose Grafana and VMAuth endpoints via Traefik
kubectl apply -f 8-ingress.yaml