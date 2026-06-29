# Backend - Tienda de Perritos 🐶

Este repositorio contiene la API (Backend) para el proyecto **Tienda de Perritos** de Innovatech Chile.
Se conecta a una base de datos MySQL para gestionar los datos de la tienda, y está preparado para desplegarse en AWS EKS.

## 🚀 Requisitos Previos

- Node.js (para desarrollo local).
- Docker instalado.
- Base de datos MySQL.
- AWS CLI y `kubectl` configurados con tu clúster de EKS.

## 🛠️ Tecnologías

- Node.js y Express.
- Conector MySQL.
- Docker.
- Kubernetes (EKS).

## ⚙️ Configuración y Despliegue Local

1. **Instalar dependencias:**
   ```bash
   npm install
   ```

2. **Variables de Entorno:**
   Asegúrate de configurar las variables de entorno necesarias (ej. host, usuario, password de DB) que el `server.js` espera recibir.

3. **Ejecutar el servidor localmente:**
   ```bash
   node server.js
   ```

4. **Ejecutar usando Docker:**
   ```bash
   docker build -t backend-tienda .
   docker run -d -p 3000:3000 -e DB_HOST=tu_host -e DB_USER=tu_usuario -e DB_PASSWORD=tu_password backend-tienda
   ```

## ☁️ Despliegue en AWS EKS

El ciclo CI/CD está manejado a través de **GitHub Actions**. Para desplegar en Kubernetes:

1. **Configurar la conexión al clúster:**
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name <NOMBRE_TU_CLUSTER>
   ```

2. **Aplicar los Secrets y Base de Datos (previo al backend):**
   ```bash
   kubectl apply -f k8s/mysql-secret.yaml
   kubectl apply -f k8s/mysql-deployment.yaml
   kubectl apply -f k8s/mysql-service.yaml
   ```

3. **Despliegue manual del Backend:**
   ```bash
   kubectl apply -f k8s/backend-deployment.yaml
   kubectl apply -f k8s/backend-service.yaml
   kubectl apply -f k8s/backend-hpa.yaml
   ```

4. **Verificar los Logs:**
   ```bash
   kubectl logs -f deployment/backend-deployment -n tienda
   ```

## 📈 Autoscaling

Este componente tiene configurado un **Horizontal Pod Autoscaler (HPA)** que escala los pods del backend automáticamente en caso de detectar alta demanda de CPU o memoria, garantizando disponibilidad y tolerancia a fallos.
