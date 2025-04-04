# Documentación del Sistema en GCP

## 1. Introducción
- **Descripción general del sistema**:  
  ...
  
- **Diagrama de arquitectura**:  
  ![Diagrama de Arquitectura](./diagrama.png)  

## 2. Estructura de Repositorios
### Frontend
- **Descripción**:  
  ...
- **Tecnología utilizada**:  
  React, Angular.
- **Ubicación en GitHub**:  
  - [Loker-Mailbox-Admin](https://github.com/Chazki-Unified/Loker-Mailbox-Admin)  
  - [Loker-Admin-Web](https://github.com/Chazki-Unified/Loker-Admin-Web)  
  - [Loker-p2p-admin](https://github.com/Chazki-Unified/Loker-p2p-admin)  
  - [Loker-iot-Admin](https://github.com/Chazki-Unified/Loker-iot-Admin)  

- **Instrucciones**:
  - Clonar repositorio:  
    ```bash
    git clone <repositorio-url>
    ```
  - Instalar dependencias:  
    ```bash
    npm install
    ```
  - Ejecutar localmente:  
    ```bash
    npm start
    ```
- **Enlaces a versiones desplegadas**:  
  - [Mailbox](https://lockers-mailboxadmin.chazki.com/)
  - [Companies](https://lockers-companies.chazki.com/)
  - [P2padmin](https://lockers-p2padmin.chazki.com/)
  - [Iotadmin](https://lockers-iotadmin.chazki.com/)

### Backend (Microservicios)
- **Descripción**:  
  ...
- **Tecnología utilizada**:  
  Node.js.
- **Ubicación en GitHub**:  
  - [Gateway](https://lockers-gateway.chazki.com/)  
- **Instrucciones**:
  - Clonar repositorio:  
    ```bash
    git clone <repositorio-url>
    ```
  - Instalar dependencias:  
    ```bash
    npm install 
    ```
  - Ejecutar localmente:  
    ```bash
    npm run start 
    ```
- **Enlaces a versiones desplegadas**:  
  - [Gateway](https://lockers-gateway.chazki.com/)

### MQTT Stream
- **Descripción**:  
  Sistema de transmisión de datos mediante MQTT.
- **Tecnologías involucradas**:  
  EMQX v5.0.26.
- **Repositorio en GitHub**:  
  - [Loker-Mqtt](https://github.com/Chazki-Unified/Loker-Mqtt)
- **Instrucciones**:
  - Clonar repositorio:  
    ```bash
    git clone <repositorio-url>
    ```
  - Instalar dependencias:  
    ```bash
    npm install 
    ```
  - Ejecutar localmente:  
    ```bash
    npm run start
    ```
- **Enlaces a versiones desplegadas**:  
  - [Mqtt](https://lockers-mqtt.chazki.com/)

## 3. Automatización del Despliegue en GCP
### Descripción de los Triggers en GCP
- Triggers configurados en GCP para automatizar el despliegue.
- **Ejemplo de configuración de un trigger**:  
  Los triggers se activan con los siguientes eventos de GitHub:
  - Push a ramas específicas (por ejemplo, `main`, `beta`, `develop`)
  - Creación de una nueva etiqueta (tag) o release

### Generación de Imágenes de los Repositorios
- **Proceso de creación de imágenes**:  
  Se utiliza **Cloud Build** para crear imágenes Docker a partir de los repositorios de frontend, backend y MQTT.
- **Herramientas utilizadas**:  
  Docker, Cloud Build, Dockerfile

### Despliegue en el Cluster
- **Despliegue en Google Kubernetes Engine (GKE)**:  
  Una vez generadas las imágenes, los triggers despliegan las imágenes en un clúster de GKE utilizando Kubernetes.
- **Estrategias de despliegue**:  
  - Rolling Updates
  - Canary Releases

- **Instrucciones para verificar despliegue**:
  - Verificar los logs de GKE:
    ```bash
    gcloud logging read "resource.type=k8s_cluster"
    ```

## 4. Configuración de la Infraestructura en GCP
### Recursos Utilizados en GCP
- **Servicios de GCP**:
  - Google Kubernetes Engine (GKE)
  - Cloud Build
  - Cloud Logging
  - Container Registry / Artifact Registry

- **Configuración de Red y Seguridad**:
  - Firewalls
  - VPCs
  - IAM Roles

## 5. Procedimiento de Despliegue Manual (si aplica)
- **Proceso Manual de Despliegue**:
  En caso de que los triggers automáticos fallen, puedes seguir este proceso manual:
  - Construir imágenes Docker:
    ```bash
    docker build -t <image-name> .
    ```
  - Subir las imágenes a Container Registry:
    ```bash
    docker push <image-name>
    ```
  - Desplegar las imágenes en GKE:
    ```bash
    kubectl apply -f deployment.yaml
    ```

## 6. Monitoreo y Logs
### Monitoreo de los Servicios
- **Herramientas de monitoreo**:
  - Cloud Monitoring
  - Prometheus
  - Grafana
  
- **Configuración de alertas**:
  - Notificaciones por fallos de microservicios, errores 5xx, o métricas críticas.

### Revisión de Logs
- **Acceso a Logs**:
  - **Logs de GKE**:
    ```bash
    gcloud logging read "resource.type=k8s_cluster"
    ```

## 7. Escalabilidad y Optimización
### Escalabilidad del Backend
- **Escalado automático en GKE**:  
  Los microservicios están configurados para escalar automáticamente según las métricas definidas.

### Optimización de Costos
- **Estrategias de optimización**:
  - Uso eficiente de recursos.
  - Pausar servicios no utilizados durante las horas no laborales.

## 8. Mantenimiento y Actualizaciones
### Estrategia de Mantenimiento
- **Actualización de microservicios o frontend**:  
  Si es necesario actualizar los microservicios o frontend, puedes seguir el flujo de CI/CD para actualizar los contenedores sin interrupciones.

### Actualización de Dependencias
- **Manejo de actualizaciones**:
  - Actualizar las dependencias utilizando los comandos correspondientes en frontend y backend.
  - Probar localmente y crear nuevas versiones de las imágenes antes de desplegar en GKE.

## 9. Resumen de Contacto y Soporte
- **Soporte Técnico**:
  - Contactar a soporte: [soporte@chazki.com](mailto:soporte@chazki.com)
  - Canal de Slack: `#pd-squad-lockers`

---

## Enlaces Útiles
- [Documentación de GKE](https://cloud.google.com/kubernetes-engine/docs)
- [Documentación de Cloud Build](https://cloud.google.com/build/docs)
- [Repositorios de GitHub](https://github.com/Chazki-Unified)
