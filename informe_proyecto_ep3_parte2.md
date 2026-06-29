# Informe de Configuración y Despliegue - EP3 (Parte 2)
**Alumno:** [Nombre del Alumno]
**Asignatura:** Introducción a Herramientas DevOps

## 9. Integración y Despliegue Continuo (CI/CD) - IE4
Para automatizar el proceso de construcción e inyección de la aplicación, se implementó un pipeline de CI/CD utilizando GitHub Actions. El flujo de trabajo (`deploy-eks.yml`) fue configurado para ejecutarse ante cada actualización en la rama principal (`main`).
1. Se almacenaron las credenciales de AWS de forma segura utilizando los *Secrets* del repositorio en GitHub.
2. El pipeline realiza la autenticación con AWS, compila las imágenes Docker, las sube a ECR y aplica los manifiestos de Kubernetes en el clúster.

>[ INSERTE CAPTURA AQUÍ: Ve a tu repositorio en GitHub > Pestaña "Actions" > Haz clic en la ejecución exitosa (ticket verde) y toma captura mostrando todos los pasos listos. ]
> **Figura 13:** Ejecución exitosa del pipeline de integración y despliegue continuo en GitHub Actions. Se evidencia la correcta construcción y despliegue automatizado de los microservicios hacia Amazon EKS.

>[ INSERTE CAPTURA AQUÍ: Ve a GitHub > Settings > Secrets and variables > Actions y toma captura a la lista de secretos configurados. ]
> **Figura 14:** Variables sensibles y credenciales de AWS resguardadas de manera segura mediante GitHub Secrets, necesarias para la autenticación del flujo de trabajo automatizado.

## 10. Gestión de Variables Sensibles (Secrets) - IE5
La seguridad interna del clúster se manejó a través de objetos *Secret* de Kubernetes, aislando la información confidencial de la base de datos (contraseña de root) fuera de los manifiestos de despliegue regulares o del código fuente.

>[ INSERTE CAPTURA AQUÍ: En tu terminal, ejecuta `kubectl get secret -n tienda` y toma captura. ]
> **Figura 15:** Implementación de Kubernetes Secrets en el espacio de nombres de la aplicación. Estos objetos proveen a los contenedores las variables de entorno necesarias para la conexión a la base de datos de forma encriptada.

## 11. Monitoreo de Registros (Logs) - IE6
Para garantizar la observabilidad y facilitar la detección de errores de los microservicios, se implementó la revisión de registros (logs) del sistema. Se comprobó que tanto el Backend como el Frontend no presentaran excepciones críticas durante el arranque.

>[ INSERTE CAPTURA AQUÍ: En tu terminal, ejecuta `kubectl get pods -n tienda`, copia el nombre de uno de los pods del backend y luego ejecuta `kubectl logs <nombre-del-pod> -n tienda`. Toma captura. ]
> **Figura 16:** Visualización de los registros (Logs) del microservicio Backend. Se confirma que la API de la tienda inicia correctamente y logra establecer conexión con la base de datos sin presentar errores.

>[ INSERTE CAPTURA AQUÍ: Ejecuta `kubectl logs <nombre-del-pod-frontend> -n tienda` y toma captura. ]
> **Figura 17:** Registros operativos del servidor Frontend. La ausencia de errores críticos de sistema confirma que el servicio web está listo para procesar peticiones HTTP.

## 12. Validación y Resiliencia del Sistema - IE7
Finalmente, para validar la resiliencia y alta disponibilidad proveída por la orquestación de Kubernetes, se realizó una prueba de recuperación de desastres (Chaos Testing básico). Se eliminó manualmente un pod de la aplicación para confirmar que el controlador interviene y levanta una nueva réplica automáticamente.

>[ INSERTE CAPTURA AQUÍ: Ejecuta `kubectl delete pod <cualquier-pod-del-frontend> -n tienda`. Inmediatamente después ejecuta `kubectl get pods -n tienda` y toma captura donde se vea que uno se está eliminando ("Terminating") y otro está creando ("ContainerCreating"). ]
> **Figura 18:** Prueba de resiliencia del orquestador. Al forzar la eliminación de un pod activo, el controlador de Kubernetes (*ReplicaSet*) detecta la discrepancia con el estado deseado y automáticamente aprovisiona una nueva instancia para mantener la disponibilidad del servicio.
