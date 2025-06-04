# 📊 Monitoreo con Prometheus y Grafana

Este archivo configura:
- Un `ServiceMonitor` que permite a Prometheus scrapear métricas desde `/actuator/prometheus`.
- Apunta al servicio de PetClinic (puerto 8080).
- Necesita estar en el mismo namespace donde está el Prometheus Operator (`monitoring`).

✅ Requiere Helm chart de kube-prometheus-stack previamente instalado.
