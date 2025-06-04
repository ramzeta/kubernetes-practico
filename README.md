# Spring PetClinic + Prometheus + Grafana + Kubernetes Setup

## 🧑‍💻 Introducción para Desarrolladores

Esta guía está diseñada para desarrolladores que buscan entender cómo desplegar aplicaciones con monitoreo en Kubernetes usando Spring PetClinic, Prometheus y Grafana. Se incluyen conceptos clave, arquitectura de Kubernetes y principios SOLID aplicados al código.

---

## 🧠 Fundamentos de Kubernetes para Desarrolladores

### Conceptos Clave

* **Pods**: Unidad mínima de ejecución.
* **Deployments**: Gestiona actualizaciones y réplicas.
* **Services**: Exponen pods dentro o fuera del cluster.
* **ConfigMap/Secrets**: Configuración externa y datos sensibles.

### Comandos Útiles

```bash
kubectl get pods
kubectl logs <pod>
kubectl exec -it <pod> -- bash
kubectl apply -f archivo.yaml
```

### Buenas prácticas

* Usa probes (`liveness`, `readiness`).
* Usa límites y requests de recursos.
* Versiona tus YAMLs.
* Usa `helm` charts en proyectos complejos.

---

## 🧱 Arquitectura de Kubernetes (Explicada)

### 1. Plano de Control (Control Plane)

* **etcd**: Base de datos distribuida del cluster.
* **Controller Manager**: Supervisa y mantiene el estado deseado.
* **Scheduler**: Asigna pods a nodos según disponibilidad de recursos.

### 2. Nodos Worker

* **Pods**: Ejecutan tu app.
* **Container Runtime**: Docker, containerd, etc.
* **kubelet**: Comunica el nodo con el plano de control.
* **System Services**: Servicios OS como red, almacenamiento.

### 3. Red y exposición

* **Load Balancer**: Expone la app al exterior.
* **Usuarios Finales**: Acceden vía navegador o cliente.

### Flujo de despliegue

1. Defines un Deployment con YAML.
2. Scheduler lo asigna a un nodo.
3. Kubelet lo lanza como pod.
4. Load Balancer expone el servicio.

### YAML de ejemplo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: grafana
```

---

## 🚀 Despliegue Paso a Paso

### 1. Iniciar Minikube

```bash
minikube start --driver=docker
```

### 2. Desplegar base de datos y aplicación PetClinic

```bash
kubectl apply -f k8s/db.yml
kubectl apply -f k8s/petclinic.yml
```

### 3. Instalar Prometheus + Grafana con Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

### 4. Configurar el monitoreo personalizado de PetClinic

```bash
kubectl apply -f k8s/petclinic-monitoring.yaml
```

### 5. Verificar que Prometheus scrapea `/actuator/prometheus`

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
  endpoint:
    prometheus:
      enabled: true
```

### 6. Exponer servicios localmente

```bash
kubectl port-forward svc/petclinic 8080:8080
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9090:9090
```

### 7. Acceder a Grafana

* URL: [http://localhost:3000](http://localhost:3000)
* Usuario: `admin`
* Contraseña:

```bash
kubectl get secret -n monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

### 8. Añadir Prometheus como Data Source en Grafana

* URL: `http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090`

---

## 📈 Monitoreo con Prometheus y Grafana

### Prometheus

* Usa `ServiceMonitor` o `scrape_configs` personalizados:

```yaml
scrape_configs:
  - job_name: 'mi-app'
    static_configs:
      - targets: ['mi-app-service:8080']
```

### Grafana

* Datasource (Prometheus):

```yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-service:9090
```

### Persistencia y Seguridad

```yaml
volumes:
  - name: grafana-storage
    persistentVolumeClaim:
      claimName: grafana-pvc
resources:
  limits:
    cpu: "1"
    memory: "1Gi"
  requests:
    cpu: "0.5"
    memory: "512Mi"
```

---

## 🔄 Reiniciar desde cero (Cleanup)

```bash
kubectl delete -f k8s/db.yml
kubectl delete -f k8s/petclinic.yml
kubectl delete -f k8s/petclinic-monitoring.yaml
helm uninstall kube-prometheus-stack -n monitoring
kubectl delete namespace monitoring
kubectl get all -A
kubectl get namespaces
```

---

## 📐 Principios SOLID (con ejemplos Java)

### 🟡 SRP – Single Responsibility Principle

```java
public class UserService {
  private final UserRepository userRepository;
  public void registerUser(User user) {
    validate(user);
    userRepository.save(user);
  }
}
```

### 🟢 OCP – Open/Closed Principle

```java
public interface PaymentMethod { void pay(); }
public class CreditCardPayment implements PaymentMethod { public void pay() {} }
public class PaypalPayment implements PaymentMethod { public void pay() {} }
```

### 🔵 LSP – Liskov Substitution Principle

```java
public interface Bird {}
public interface FlyingBird extends Bird { void fly(); }
```

### 🔴 ISP – Interface Segregation Principle

```java
public interface Printer { void print(Document doc); }
public interface Scanner { void scan(Document doc); }
```

### 🔸 DIP – Dependency Inversion Principle

```java
public interface NotificationService { void notify(String message); }
@Service
public class EmailNotificationService implements NotificationService {
  public void notify(String message) { }
}
```

---

## 📘️ Recursos Recomendados

* [https://kubernetes.io/docs/tutorials/kubernetes-basics/](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
* [https://killercoda.com/](https://killercoda.com/)
* [https://grafana.com/docs/](https://grafana.com/docs/)
* [https://prometheus.io/docs/introduction/overview/](https://prometheus.io/docs/introduction/overview/)
* [https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)
* [https://artifacthub.io/packages/helm/grafana/grafana](https://artifacthub.io/packages/helm/grafana/grafana)
