# 📦 PostgreSQL para PetClinic

Este archivo define:
- Un `Secret` con la contraseña codificada en base64.
- Un `PersistentVolumeClaim` de 1Gi.
- Un `Deployment` con configuración de probes de salud.
- Un `Service` que expone el puerto `5432`.

✅ La base de datos es persistente y segura.  
⚠️ Asegúrate de crear el PVC antes o usar almacenamiento dinámico.
