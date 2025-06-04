
# Tutorial: Despliegue de Spring PetClinic con Kubernetes, Prometheus y Grafana

## 🛠 Requisitos previos
- Docker y Minikube instalados.
- `kubectl`, `helm` y `k9s` (opcional).
- Clonaste: https://github.com/spring-projects/spring-petclinic

## 📁 Estructura esperada
```
spring-petclinic/
├── k8s/
│   ├── db.yml
│   ├── petclinic.yml
│   └── petclinic-monitoring.yaml
```

## 🚀 1. Iniciar Minikube
```bash
minikube start --cpus=4 --memory=8192 --driver=docker
```

## 🐘 2. Desplegar PostgreSQL
```bash
kubectl apply -f k8s/db.yml
kubectl get pods -l app=postgres
```

## 🐶 3. Desplegar Spring PetClinic
```bash
kubectl apply -f k8s/petclinic.yml
kubectl get pods -l app=petclinic
```

## 🌐 4. Acceder a la app
```bash
minikube service petclinic
```

## 📊 5. Añadir Prometheus y Grafana
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

## ⚙️ 6. Aplicar configuración personalizada
```bash
kubectl apply -f k8s/petclinic-monitoring.yaml
```

## 🔒 7. Acceder a Grafana
```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```
Abrir [http://localhost:3000](http://localhost:3000) con admin/prom-operator

## 📈 8. Añadir panel de métricas de Spring Boot
Importar ID `4701` en Grafana

## 📄 9. Explicación de YAMLs
- `db.yml`: PostgreSQL
- `petclinic.yml`: Spring App
- `petclinic-monitoring.yaml`: ServiceMonitor

## 🧪 10. Verificación
```bash
kubectl get all
```
