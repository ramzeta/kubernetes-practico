# 🐶 Aplicación Spring PetClinic

Incluye:
- `Deployment` con una instancia.
- Variables de entorno con config JSON para base de datos y Prometheus.
- `readinessProbe` y `livenessProbe` basadas en endpoints de Actuator.
- `Service` etiquetado con `release: kube-prometheus-stack` para integración con Prometheus.

⚠️ Asegúrate de tener habilitado `/actuator/prometheus` para recolectar métricas.
