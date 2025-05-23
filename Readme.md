# 🧭 EKS Monitoring Stack

This repository provides a complete observability stack for Amazon EKS using:

- **Prometheus** – Metrics
- **Grafana** – Dashboards & visualization
- **Loki + Promtail** – Log aggregation
- **Tempo** – Distributed tracing backend
- **OpenTelemetry Collector** – Unified pipeline for telemetry data
- **HotROD App** – A demo app to generate metrics, logs, and traces

It includes support for metrics, logs, and traces using Helm charts and Kubernetes YAML manifests.

---

## 📁 Repository Structure

```
.
│── otel-collector.yaml     # OpenTelemetry Collector deployment
│── hotrod.yaml             # HotROD demo app
|── loki-values.yaml        # loki setup
|── loki-storage.yaml       # loki storage deployment
|── tempo-values.yaml       # Monolith tempo deployment
|── tempo-distributor.yaml. # Microservice tempo deployment
|── ingress.yaml            # Ingress for inbound alb
|── promtail-values.yaml.   # Promtail send additional logs to loki
|── README.md                   
```

---

## ⚙️ Prerequisites

- EKS cluster
- `kubectl` configured
- `helm` installed
- `aws` CLI configured (optional)
- Namespace: `monitoring`

Create the namespace (if not already):

```bash
kubectl create namespace monitoring
```

---

## 🚀 Installation Steps

### 1. Add Helm Repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

---

### 2. Install Prometheus & Grafana

```bash
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack   --namespace monitoring --create-namespace
```
> This sets up Prometheus, Alertmanager, Node Exporter, Kube State Metrics, and Grafana.

 
---

### 3. Install Loki and Promtail

```bash
helm upgrade --install loki grafana/loki-stack   --namespace monitoring
```

---

### 4. Install Tempo (Traces backend)

```bash
helm upgrade --install tempo grafana/tempo   --namespace monitoring
```

---

### 5. Deploy OpenTelemetry Collector

```bash
kubectl apply -f otel-collector.yaml
```

> This Collector receives OTLP telemetry and routes to Prometheus, Loki, and Tempo.

---

### 6. Deploy HotROD Demo App

```bash
kubectl apply -f hotrod.yaml
```

> The HotROD app will send metrics, logs, and traces using OpenTelemetry.

---

## 📊 Access Grafana

1. Port-forward:

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80
```

2. Open browser: [http://localhost:3000](http://localhost:3000)

3. Login Credentials:

- Username: `admin`
- Password: run the following to get it:

```bash
kubectl get secret --namespace monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

---

## 🔌 Grafana Data Sources

Helm auto-configures:

- Prometheus → Metrics
- Loki → Logs
- Tempo → Traces

If needed, manually add in Grafana under **Settings > Data Sources**.

---

## ✅ Validation

- **Metrics** → Use Prometheus metrics explorer in Grafana
- **Logs** → View logs by pod or label using Loki
- **Traces** → Explore traces using Tempo’s integration in Grafana

---

## 🧹 Cleanup

```bash
helm uninstall kube-prometheus-stack -n monitoring
helm uninstall loki -n monitoring
helm uninstall tempo -n monitoring
kubectl delete -f <.yaml file>
```

---

## 📚 References

- [Prometheus Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Loki Stack](https://grafana.com/docs/loki/latest/installation/helm/)
- [Tempo Helm Chart](https://grafana.com/docs/tempo/latest/)
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
- [HotROD App](https://github.com/jaegertracing/jaeger/tree/main/examples/hotrod)

---

