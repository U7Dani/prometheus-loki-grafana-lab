![image](https://github.com/user-attachments/assets/b6e76d57-7e68-46fe-8e2b-8be3a263b1bc)

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
![Captura de pantalla 2025-05-25 134910](https://github.com/user-attachments/assets/5abaf044-89e2-4aa4-b88c-31ebe1dc9f0b)

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
![Captura de pantalla 2025-05-25 134317](https://github.com/user-attachments/assets/34fbc36c-2380-4d30-bd09-3d7335d5b610)
![Captura de pantalla 2025-05-25 134501](https://github.com/user-attachments/assets/d35bfea6-aa96-4095-b5e1-10f8d3814f14)


---

## 🚨 ¿Qué tipo de alertas se pueden crear?

- Alto % de errores HTTP (5xx)
- Latencia superior a 500ms en endpoints críticos
- Exceso de uso de CPU/memoria
- Picos de actividad inusual (detección de amenazas)

Se definen en Prometheus (`rules`) o desde Grafana (alertas visuales).

![Captura de pantalla 2025-05-25 134632](https://github.com/user-attachments/assets/89183eb5-e5d3-45b7-810f-1ebfa205e06c)

📸 Dashboard principal en Grafana: Application Monitoring & Logging
Esta captura muestra el dashboard principal integrado en Grafana, donde se visualizan en tiempo real las métricas clave de una aplicación:

📊 Paneles destacados:
🔁 Request Rate by Method
Muestra el número de peticiones por segundo (req/s) clasificadas por método HTTP (GET, DELETE, HFAD, etc.). Ideal para detectar incrementos súbitos de tráfico o cambios inesperados en la distribución de métodos.

📈 Status Code Distribution
Diagrama circular con los códigos HTTP devueltos (2xx, 4xx, 5xx...). Una alta proporción de errores (404, 503) puede indicar fallos en endpoints o incidentes activos.

⏱️ Response Time by Method (p50 y p95)
Medición de la latencia media (p50) y de alta carga (p95) por método. Permite identificar degradaciones de rendimiento en rutas específicas.

📉 Status Codes Over Time
Evolución temporal de los códigos HTTP devueltos. Ayuda a detectar cuándo comenzaron a producirse errores, facilitando el análisis forense o la correlación con eventos externos.

![Captura de pantalla 2025-05-25 134746](https://github.com/user-attachments/assets/75bf2f7b-cb21-4b0e-95ec-90258073703f)

📸 Análisis por volumen y latencia de endpoints
Esta sección del dashboard ofrece una vista centrada en el tamaño de respuesta, la latencia y el volumen de tráfico por endpoint. Es útil para identificar cuellos de botella, endpoints lentos y puntos calientes de uso.

📦 Paneles destacados:
📐 Response Size by Method
Tamaño promedio de las respuestas HTTP según el método (GET, DELETE, HFAD).
Útil para detectar:

Respuestas inesperadamente grandes (p. ej. fugas de datos).

Cambios de comportamiento tras despliegues.

🐢 Top 10 Slowest Endpoints (p95)
Lista de rutas que, en el 95% de las veces, tienen mayor latencia.
Ideal para enfocar esfuerzos de optimización o detección de recursos críticos.

🔥 Top 10 Endpoints by Request Volume
Muestra los endpoints con más tráfico.
Su análisis permite priorizar:

Endpoints más expuestos a errores o abusos.

Objetivos potenciales en escenarios de ataque.






---

## 📌 Ideas para el futuro

- Integración con **Wazuh** como SIEM
- Exportar métricas/logs a **Elasticsearch** o **MISP**
- Detección de anomalías con IA o Reglas
- Dashboards por tipo de incidente (seguridad, rendimiento)
- Alertas con **Slack**, **Telegram**, **Email**


---

## 📜 Licencia

MIT License

---

## ✍️ Autor

Daniel Utrilla  
🔗 GitHub: [github.com/U7Dani](https://github.com/U7Dani)  
🔗 LinkedIn: [linkedin.com/in/danielutrilla](https://linkedin.com/in/danielutrilla)
