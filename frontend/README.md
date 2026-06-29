# Frontend - Tienda de Perritos 🐶

Este repositorio contiene el código fuente del Frontend para el proyecto **Tienda de Perritos** de Innovatech Chile.
Esta aplicación ha sido diseñada para ser contenerizada mediante Docker y desplegada en un clúster de Kubernetes (AWS EKS).

## 🚀 Requisitos Previos

- Docker instalado.
- AWS CLI y `kubectl` configurados con el clúster EKS.
- Acceso al registro de contenedores (Amazon ECR).

## 🛠️ Tecnologías

- HTML / CSS / Vanilla JS (o Framework correspondiente si aplica).
- Nginx (usado como servidor web en el contenedor Docker).
- Docker.
- Kubernetes (EKS).

## ⚙️ Configuración y Despliegue Local (Docker)

1. **Construir la imagen:**
   ```bash
   docker build -t frontend-tienda .
   ```

2. **Ejecutar el contenedor:**
   ```bash
   docker run -d -p 8080:80 frontend-tienda
   ```

3. **Acceder a la aplicación:**
   Abre tu navegador en `http://localhost:8080`.

## ☁️ Despliegue en AWS EKS

El despliegue está automatizado mediante **GitHub Actions** (CI/CD), pero los manifiestos de Kubernetes se encuentran en la carpeta `k8s/` del repositorio principal.

1. **Configurar contexto del clúster:**
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name <NOMBRE_TU_CLUSTER>
   ```

2. **Despliegue manual (si es necesario):**
   ```bash
   kubectl apply -f k8s/frontend-deployment.yaml
   kubectl apply -f k8s/frontend-service.yaml
   kubectl apply -f k8s/frontend-hpa.yaml
   ```

3. **Verificar el servicio:**
   ```bash
   kubectl get svc frontend-service -n tienda
   ```
   *Copia la URL pública (EXTERNAL-IP) del Load Balancer y ábrela en tu navegador.*

## 📈 Autoscaling

Este componente tiene configurado un **Horizontal Pod Autoscaler (HPA)** que monitorea la carga (CPU/Memoria) y escala los pods automáticamente para mantener la disponibilidad según los parámetros en `frontend-hpa.yaml`.
