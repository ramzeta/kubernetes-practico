
# Spring PetClinic + Prometheus + Grafana + Kubernetes Setup

## 🏁 Requisitos
- Minikube
- Helm
- kubectl
- Java 17 y Gradle (para compilar el backend)
- Docker
- WSL o entorno Linux/Unix para ejecutar scripts opcionales

## 🚀 Despliegue Paso a Paso

### 1. Iniciar Minikube
```bash
minikube start --driver=docker
```

### 2. Desplegar la base de datos y aplicación
```bash
kubectl apply -f k8s/db.yml
kubectl apply -f k8s/petclinic.yml
```

### 3. Instalar Prometheus y Grafana con Helm
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

### 4. Añadir monitoreo de PetClinic
```bash
kubectl apply -f k8s/petclinic-monitoring.yaml
```

### 5. Exponer servicios localmente
```bash
kubectl port-forward svc/petclinic 8080:8080
kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80
kubectl port-forward svc/kube-prometheus-stack-prometheus -n monitoring 9090:9090
```

### 6. Acceder a Grafana
- URL: http://localhost:3000
- Usuario: `admin`
- Contraseña: obtener con:
```bash
kubectl get secret -n monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

### 7. Añadir Data Source (Prometheus) en Grafana:
- URL: `http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090`

---

## 🧹 Cleanup - Eliminar todo para empezar desde cero

```bash
# Eliminar PetClinic y sus monitorizaciones
kubectl delete -f k8s/db.yml
kubectl delete -f k8s/petclinic.yml
kubectl delete -f k8s/petclinic-monitoring.yaml

# Eliminar Prometheus y Grafana
helm uninstall kube-prometheus-stack -n monitoring

# Eliminar namespace monitoring
kubectl delete namespace monitoring

# Comprobar que está limpio
kubectl get all -A
kubectl get namespaces
```

---

## 🔠 Principios SOLID

---

### 🟡 **S – Single Responsibility Principle (SRP)**

**Una clase debe tener una única razón para cambiar.**

📌 *Cada clase o módulo debe encargarse de una única responsabilidad dentro del sistema.*

#### ✅ Ejemplo en Spring Boot:

```java
// 👎 Mal: mezclando lógica de negocio y persistencia
public class UserService {
    public void registerUser(User user) {
        validate(user);
        saveToDatabase(user); // lógica de persistencia
    }
}

// 👍 Bien: separación de responsabilidades
public class UserService {
    private final UserRepository userRepository;

    public void registerUser(User user) {
        validate(user);
        userRepository.save(user); // delega la persistencia
    }

    private void validate(User user) {
        // validaciones
    }
}
```

---

### 🟢 **O – Open/Closed Principle (OCP)**

**El código debe estar abierto a la extensión, pero cerrado a la modificación.**

📌 *Puedes añadir nueva lógica sin tener que modificar clases existentes.*

#### ✅ Ejemplo en Java con Strategy:

```java
public interface PaymentMethod {
    void pay();
}

public class CreditCardPayment implements PaymentMethod {
    public void pay() {
        // lógica de tarjeta
    }
}

public class PaypalPayment implements PaymentMethod {
    public void pay() {
        // lógica de PayPal
    }
}

public class PaymentProcessor {
    public void process(PaymentMethod method) {
        method.pay(); // se puede extender sin tocar el processor
    }
}
```

---

### 🔵 **L – Liskov Substitution Principle (LSP)**

**Una clase hija debe poder sustituir a su clase padre sin romper el comportamiento.**

📌 *Si una subclase no puede comportarse como su clase padre, no está bien diseñada.*

#### ✅ Ejemplo correcto:

```java
public class Bird {
    public void fly() {}
}

public class Eagle extends Bird {
    public void fly() {
        System.out.println("Eagle flying");
    }
}
```

#### ❌ Ejemplo incorrecto (rompe LSP):

```java
public class Penguin extends Bird {
    public void fly() {
        throw new UnsupportedOperationException(); // los pingüinos no vuelan
    }
}
```

🛠 Solución: crear jerarquías adecuadas.

```java
public interface Bird {}

public interface FlyingBird extends Bird {
    void fly();
}
```

---

### 🔴 **I – Interface Segregation Principle (ISP)**

**Los clientes no deberían depender de interfaces que no usan.**

📌 *Es mejor tener interfaces pequeñas y específicas.*

#### ✅ Ejemplo:

```java
public interface Printer {
    void print(Document doc);
}

public interface Scanner {
    void scan(Document doc);
}
```

📌 En vez de:

```java
public interface Machine {
    void print();
    void scan();
    void fax(); // no todos usan fax
}
```

---

### 🟣 **D – Dependency Inversion Principle (DIP)**

**Depende de abstracciones, no de implementaciones concretas.**

📌 *Las clases de alto nivel no deben depender de clases de bajo nivel directamente.*

#### ✅ Ejemplo en Spring:

```java
public interface NotificationService {
    void notify(String message);
}

@Service
public class EmailNotificationService implements NotificationService {
    public void notify(String message) {
        // enviar email
    }
}

@Service
public class UserService {
    private final NotificationService notificationService;

    @Autowired
    public UserService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

👉 Esto permite cambiar fácilmente `EmailNotificationService` por `SMSNotificationService`, por ejemplo.

---
# kubernetes-practico
