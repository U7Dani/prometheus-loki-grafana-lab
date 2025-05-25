
# 📦 prometheus-loki-grafana-lab

## 📈 Observabilidad en Kubernetes con Prometheus, Loki y Grafana

Este proyecto despliega un stack completo de **observabilidad sobre Kubernetes local (Minikube)**, ideal como punto de partida para entornos DevOps, Blue Team o laboratorios de detección de amenazas.

Incluye:
- 📦 Recolección de **métricas** con Prometheus
- 🔍 Ingesta de **logs** con Loki + Promtail
- 📊 Visualización avanzada con Grafana
- 🧪 Generación de logs y métricas simuladas para pruebas
- ⚠️ Soporte para alertas y futuras integraciones con SIEM

---

## 🚀 Requisitos

- Ubuntu 22.04 o superior (funciona en VM)
- Docker (activo y funcional)
- kubectl
- Helm
- Minikube (con driver Docker)

---

## ⚙️ Instalación paso a paso

### 1. Instalar dependencias (una sola vez)

```bash
sudo apt update && sudo apt install -y curl docker.io
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
sudo snap install helm --classic
```

### 2. Iniciar Minikube

```bash
minikube start --driver=docker --memory=4096 --cpus=2
```

---

## 📁 Estructura del repositorio

```
.
├── grafana/
│   └── grafana-dashboard.json
├── loki/
│   └── loki-values.yaml
├── prometheus/
│   └── prometheus-values.yaml
├── test/
│   └── fake-metrics-logs.yaml
├── screenshots/
│   └── (capturas de ejemplo)
└── README.md
```

---

## 📦 Despliegue del Stack

### 1. Crear namespace

```bash
kubectl create namespace monitoring
```

### 2. Añadir repos Helm

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 3. Instalar Loki

```bash
helm install loki grafana/loki -n monitoring -f loki/loki-values.yaml
```

### 4. Instalar Prometheus + Grafana

> (incluye datasource Loki preconfigurado)

```bash
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring -f prometheus/prometheus-values.yaml
```

### 5. Obtener contraseña de Grafana

```bash
kubectl -n monitoring get secret prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```

### 6. Acceder a Grafana desde navegador del host

```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80 --address=0.0.0.0
```

Luego ve a: `http://<IP_de_tu_VM>:3000`  → usuario: `admin`

---

## 🧪 Cargar logs y métricas de prueba

```bash
kubectl apply -f test/fake-metrics-logs.yaml -n monitoring
```

Verifica que los pods estén corriendo:
```bash
kubectl get pods -n monitoring
```

---

## 📊 Importar Dashboard en Grafana

1. Ir a "Dashboards > Import"
2. Subir el archivo `grafana/grafana-dashboard.json`
3. Seleccionar la fuente `Prometheus` o `Loki` según se requiera

---

## 🔍 Explorar Logs en Loki

1. Ir a "Explore"
2. Elegir la fuente `Loki`
3. Usar la consulta:
```logql
{app="log-generator"}
```
4. Ver logs generados en tiempo real

---

## 🚨 ¿Qué tipo de alertas se pueden crear?

- Alto % de errores HTTP (5xx)
- Latencia superior a 500ms en endpoints críticos
- Exceso de uso de CPU/memoria
- Picos de actividad inusual (detección de amenazas)

Se definen en Prometheus (`rules`) o desde Grafana (alertas visuales).

---

## 📌 Ideas para el futuro

- Integración con **Wazuh** como SIEM
- Exportar métricas/logs a **Elasticsearch** o **MISP**
- Detección de anomalías con IA o Reglas
- Dashboards por tipo de incidente (seguridad, rendimiento)
- Alertas con **Slack**, **Telegram**, **Email**

---

## 📸 Capturas de ejemplo

Agrega aquí capturas de:
- Dashboards en Grafana
- Logs desde Loki
- Alertas activas

---

## 📜 Licencia

MIT License

---

## ✍️ Autor

Daniel Utrilla  
🔗 GitHub: [github.com/U7Dani](https://github.com/U7Dani)  
🔗 LinkedIn: [linkedin.com/in/danielutrilla](https://linkedin.com/in/danielutrilla)
