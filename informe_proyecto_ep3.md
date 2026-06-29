# Informe de Configuración y Despliegue - EP3
**Alumno:** [Nombre del Alumno]
**Asignatura:** Introducción a Herramientas DevOps

## 1. Arquitectura del Proyecto
El proyecto consiste en una aplicación de tres capas (Frontend, Backend, Base de Datos MySQL) orquestada mediante Amazon EKS. La infraestructura se diseñó utilizando dos Zonas de Disponibilidad (AZ), ubicando los Worker Nodes en subredes privadas, sin acceso directo desde el exterior, y exponiendo la aplicación únicamente mediante un Application Load Balancer (ALB) público.

## 2. Configuración de la Red (VPC y Subredes)
Para la capa de red, se realizaron los siguientes pasos en la consola de AWS:
1. Se creó una VPC personalizada con el bloque CIDR 10.0.0.0/16 utilizando la opción "VPC and more".
2. Se generaron dos subredes públicas y dos subredes privadas distribuidas en us-east-1a y us-east-1b.
3. Se habilitó un NAT Gateway en una subred pública para darle salida a internet a los nodos privados.
4. Se etiquetaron (Tags) las subredes manualmente. A las públicas se les asignó el tag `kubernetes.io/role/elb` y a las privadas `kubernetes.io/role/internal-elb`.

>[ INSERTE CAPTURA AQUÍ: Ve a VPC > Selecciona tu VPC y toma captura del "Resource Map" al final de la página. ]
> **Figura 1:** Mapa de recursos de la VPC, evidenciando la distribución equitativa de subredes en dos zonas de disponibilidad, así como las rutas de la tabla de enrutamiento hacia el NAT Gateway para garantizar el acceso a internet de las subredes privadas.

>[ INSERTE CAPTURA AQUÍ: Ve a VPC > Subnets > Selecciona una subred pública > Pestaña "Tags" y toma captura. ]
> **Figura 2:** Detalle de los *Tags* asignados a las subredes. Esta etiqueta (`kubernetes.io/role/elb`) es fundamental para que el controlador de Kubernetes reconozca en qué subredes debe desplegar automáticamente el balanceador de carga público de AWS.

## 3. Configuración de Seguridad e IAM
La seguridad se manejó a través de Security Groups:
1. Se creó el grupo `tienda-alb-sg` para el balanceador, habilitando una regla de entrada para el tráfico HTTP en el puerto 80.
2. Se creó el grupo `tienda-nodes-sg` para los nodos, configurando tres reglas de entrada:
   - Todo el tráfico interno entre los mismos nodos (All traffic).
   - El puerto HTTPS (443) para permitir la conexión del panel de control de EKS.
   - Los puertos TCP altos (1025-65535) provenientes exclusivamente del grupo del balanceador de carga.
3. Para los permisos IAM, se constató la existencia del `LabRole` predeterminado de AWS Academy, el cual se usó en todo el proyecto.

>[ INSERTE CAPTURA AQUÍ: Ve a EC2 > Security Groups > Selecciona tienda-nodes-sg > Pestaña Inbound Rules. ]
> **Figura 3:** Reglas de entrada (Inbound rules) del Security Group asignado a los Worker Nodes, confirmando la restricción de tráfico externo y la habilitación de puertos altos requeridos por el servicio tipo NodePort de Kubernetes.

## 4. Creación del Clúster EKS y Nodos
El despliegue de la infraestructura de orquestación en Amazon EKS se realizó siguiendo un proceso muy detallado en la consola de AWS:

**Fase 4.1: Creación del Clúster (Control Plane)**
1. Se accedió al servicio **Amazon EKS** y se seleccionó la opción **Add cluster > Create**.
2. En la configuración general, se le asignó el nombre al clúster y se seleccionó la versión más estable de Kubernetes (ej. 1.29).
3. En la sección de permisos (Cluster Service Role), se seleccionó el rol IAM preexistente proporcionado por el laboratorio (`LabRole` o equivalente).
4. En el apartado de redes (Networking), se seleccionó la VPC `tienda` y se asociaron tanto las subredes públicas como las privadas.
5. Se seleccionó el grupo de seguridad de los nodos (`tienda-nodes-sg`) y se configuró el acceso al endpoint como "Public and private".
6. Se finalizó el asistente y se esperó aproximadamente 15 minutos hasta que el estado del clúster cambió a **"Active"**.

>[ INSERTE CAPTURA AQUÍ: Ve a EKS > Clusters > Selecciona tu clúster y toma una foto de la pantalla principal donde sale el estado "Activo" en verde. ]
> **Figura 4:** Consola de administración de Amazon EKS, confirmando que el clúster se encuentra en estado operativo ("Active") tras finalizar el aprovisionamiento del Control Plane.

**Fase 4.2: Creación del Grupo de Nodos (Worker Nodes)**
1. Estando dentro de la vista del clúster recién creado, se accedió a la pestaña **Compute** (Informática).
2. Se hizo clic en el botón **Add Node Group** (Añadir grupo de nodos).
3. Se asignó un nombre identificativo al grupo de nodos y se seleccionó nuevamente el `LabRole` como rol IAM del nodo.
4. En la configuración de cómputo, se seleccionó una AMI de Amazon Linux 2 y el tipo de instancia `t3.medium` con un almacenamiento de 20 GB.
5. Se establecieron los parámetros de escalado: Tamaño deseado 2, Tamaño mínimo 2 y Tamaño máximo 4.
6. En el paso de redes, se seleccionaron *únicamente* las dos subredes privadas para asegurar que los nodos no queden expuestos directamente a internet.
7. Se completó la creación y se aguardó a que las instancias EC2 fuesen provisionadas.

>[ INSERTE CAPTURA AQUÍ: En la misma página de EKS, ve a la pestaña "Compute" o "Informática" y toma captura donde se vea la lista de Node Groups. ]
> **Figura 5:** Detalle del grupo de nodos (Node Group) asociado al clúster, evidenciando el despliegue exitoso de las instancias requeridas para ejecutar las cargas de trabajo.

**Fase 4.3: Conexión y Verificación de los Nodos**
1. Para gestionar el clúster localmente, se abrió la terminal y se ejecutó el comando de autenticación para descargar el Kubeconfig.
2. Finalmente, se ejecutó el comando de validación de Kubernetes para confirmar la conexión.

>[ INSERTE CAPTURA AQUÍ: En tu terminal, ejecuta `kubectl get nodes` y toma captura de los nodos. ]
> **Figura 6:** Verificación por interfaz de línea de comandos (CLI) del estado de los Worker Nodes. Se observa que los nodos se han registrado correctamente en el clúster y su estado es "Ready".

## 5. Repositorios de ECR y Subida de Imágenes
Para el manejo de los contenedores de la aplicación:
1. Se crearon tres repositorios privados en Amazon ECR (`tienda-frontend`, `tienda-backend`, `tienda-db`).
2. En la terminal local, se ejecutó el comando `docker build` para construir las tres imágenes.
3. Se etiquetaron las imágenes y se subieron (Push) a los repositorios correspondientes en AWS utilizando la terminal.

>[ INSERTE CAPTURA AQUÍ: Ve a ECR > Repositories y toma captura de la lista de tus 3 repositorios. ]
> **Figura 7:** Repositorios de imágenes de contenedores en Amazon ECR. Cada repositorio privado fue creado para almacenar de forma segura las imágenes versionadas del Frontend, Backend y Base de Datos del proyecto.

## 6. Despliegue en Kubernetes (Manifiestos y Servicios)
El despliegue final en el clúster requirió modificar el código y ejecutarlo:
1. Se editaron los archivos `.yaml` de los *Deployments* para que apuntaran a las nuevas direcciones URI de ECR.
2. Se aplicaron los manifiestos en orden utilizando el comando `kubectl apply`:
   - Primero el archivo de contraseñas (*Secrets*).
   - Luego el servicio y *deployment* de la Base de Datos.
   - Después el Backend.
   - Finalmente el Frontend (que aprovisiona automáticamente el ALB).

>[ INSERTE CAPTURA AQUÍ: En tu terminal ejecuta `kubectl get pods -n tienda` y toma captura. ]
> **Figura 8:** Estado de los Pods desplegados en el espacio de nombres de la aplicación. Todos los microservicios, incluyendo la base de datos y la interfaz de usuario, se encuentran en estado "Running".

>[ INSERTE CAPTURA AQUÍ: En tu terminal ejecuta `kubectl get svc -n tienda` y toma captura resaltando la EXTERNAL-IP del frontend. ]
> **Figura 9:** Servicios de Kubernetes aprovisionados. Se destaca la creación de un servicio tipo LoadBalancer para el Frontend, el cual expone una dirección DNS pública gestionada automáticamente por AWS para el acceso de los clientes.

## 7. Verificación del Funcionamiento de la Aplicación
Para validar que el despliegue fue exitoso:
1. Se copió la URL (EXTERNAL-IP) del balanceador en un navegador web.
2. Se realizaron pruebas insertando datos, editándolos y eliminándolos, comprobando que la interfaz se comunica sin problemas con la base de datos a través del backend.

>[ INSERTE CAPTURA AQUÍ: Abre la página de tu tienda en el navegador, agrega un producto y toma captura. ]
> **Figura 10:** Interfaz gráfica de la aplicación en funcionamiento. La visualización correcta de los datos confirma que las tres capas (Frontend, API Backend y Base de Datos) se comunican e interactúan íntegramente.

## 8. Autoescalado (HPA y Metrics Server)
La última etapa consistió en configurar la escalabilidad:
1. Se instaló Metrics Server en el clúster para habilitar el monitoreo de recursos.
2. Se verificó el funcionamiento de las métricas.
3. Se aplicaron los archivos de autoescalado (HPA), configurando el límite de CPU al 70% para el Backend y 60% para el Frontend.

>[ INSERTE CAPTURA AQUÍ: En tu terminal ejecuta `kubectl top pods -n tienda` y toma captura. ]
> **Figura 11:** Consumo de recursos de hardware de los contenedores medido por Metrics Server, validando la capacidad del clúster para recolectar métricas en tiempo real.

>[ INSERTE CAPTURA AQUÍ: En tu terminal ejecuta `kubectl get hpa -n tienda` y toma captura. ]
> **Figura 12:** Configuración del Horizontal Pod Autoscaler (HPA). Se observan los umbrales de CPU configurados (Targets) que determinarán cuándo el sistema debe escalar creando nuevas réplicas de los microservicios.
