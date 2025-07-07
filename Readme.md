# Evaluación Módulo 5: Diseño de Arquitectura y Escalabilidad
## Plataforma de Gestión de Proyectos ProManage

**Alumno:** Jesús Gabriel Castillo  
**Fecha:** Julio 2025  
**Programa:** Bootcamp DevOps  

---

## Resumen

Este informe presenta una propuesta de arquitectura de software completa para la plataforma de gestión de proyectos ProManage, implementando una arquitectura de microservicios escalable, resiliente y segura utilizando tecnologías modernas de contenedorización y orquestación en AWS.

**Decisiones Clave:**
- **Arquitectura:** Microservicios con 5 servicios principales
- **Infraestructura:** Combinación IaaS + PaaS en AWS
- **Contenedorización:** Docker con Kubernetes (EKS)
- **Escalabilidad:** Autoescalado horizontal y vertical

---

## 1. Análisis de Requerimientos y Elección de Arquitectura

### 1.1 Identificación de Componentes Clave

La plataforma ProManage requiere los siguientes componentes funcionales:

**Componentes de Dominio:**
- **Gestión de Usuarios**: Registro, autenticación, perfiles con niveles de acceso
- **Gestión de Proyectos**: CRUD completo, asignación de presupuestos
- **Gestión de Archivos**: Subida, almacenamiento y descarga de documentos
- **Gestión Temporal**: Fechas, plazos, notificaciones automatizadas
- **Gestión de Accesos**: Autorización basada en roles (RBAC)

**Componentes Técnicos:**
- **API Gateway**: Punto de entrada único y routing
- **Service Mesh**: Comunicación segura entre servicios
- **Monitoreo**: Observabilidad y métricas
- **Almacenamiento**: Base de datos y archivos
- **Cache**: Optimización de rendimiento

### 1.2 Justificación de Arquitectura: Microservicios

**Decisión: Arquitectura de Microservicios**

**Justificación Técnica:**

**Ventajas:**
- **Escalabilidad Independiente**: Cada servicio puede escalar según su demanda específica
- **Tecnología Heterogénea**: Libertad para elegir el stack tecnológico óptimo por servicio
- **Desarrollo Paralelo**: Equipos pueden trabajar independientemente sin bloqueos
- **Resiliencia**: Fallo aislado - un servicio caído no afecta todo el sistema
- **Mantenibilidad**: Código modular, deployment independiente
- **Evolución Técnica**: Fácil adopción de nuevas tecnologías por servicio

**Desventajas Mitigadas:**
- **Complejidad de Red**: Mitigada con Service Mesh (Istio)
- **Consistencia de Datos**: Patrón Saga para transacciones distribuidas
- **Monitoreo Distribuido**: Tracing distribuido con Jaeger
- **Gestión de Configuración**: ConfigMaps y Secrets en Kubernetes

**Microservicios Propuestos:**

1. **Auth Service** (Puerto 8001)
   - Autenticación JWT
   - Autorización RBAC
   - Gestión de sesiones
   - Integración OAuth2/OIDC

2. **User Service** (Puerto 8002)
   - Gestión de perfiles
   - Niveles de acceso
   - Preferencias de usuario
   - Auditoría de acciones

3. **Project Service** (Puerto 8003)
   - CRUD de proyectos
   - Gestión de presupuestos
   - Asignación de recursos
   - Reporting básico

4. **File Service** (Puerto 8004)
   - Upload/Download de archivos
   - Metadata management
   - Versionado de documentos
   - Integración con S3

5. **Notification Service** (Puerto 8005)
   - Gestión de fechas límite
   - Notificaciones email/push
   - Calendarios y recordatorios
   - Eventos del sistema

### 1.3 Elección de Infraestructura: Combinación IaaS + PaaS

**Decisión: Modelo Híbrido en AWS**

**Justificación:**

**IaaS (Infrastructure as a Service):**
- **Amazon EKS**: Control total sobre cluster Kubernetes
- **EC2 Instances**: Nodos worker personalizables
- **VPC**: Red privada virtual con control total

**PaaS (Platform as a Service):**
- **Amazon RDS**: Base de datos gestionada con backups automáticos
- **ElastiCache**: Cache gestionado Redis/Memcached
- **Application Load Balancer**: Balanceo de carga gestionado

**BaaS (Backend as a Service):**
- **Amazon S3**: Almacenamiento de objetos escalable
- **SES**: Servicio de email gestionado
- **CloudWatch**: Monitoreo y logging gestionado

**FaaS (Function as a Service):**
- **AWS Lambda**: Funciones serverless para tareas específicas
- **EventBridge**: Orquestación de eventos

**Ventajas del Modelo Híbrido:**
- **Flexibilidad**: Control donde se necesita, gestión donde es eficiente
- **Costo-Eficiencia**: Pago por uso en servicios gestionados
- **Escalabilidad**: Aprovecha el autoescalado nativo de AWS
- **Seguridad**: Servicios gestionados con seguridad enterprise
- **Mantenimiento**: Reduce overhead operacional

---

## 2. Diseño de Infraestructura y Escalabilidad

### 2.1 Estrategia de Autoescalado

**Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: project-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**Vertical Pod Autoscaler (VPA):**
- Ajuste automático de requests/limits
- Optimización de recursos por contenedor
- Análisis histórico de uso

**Cluster Autoscaler:**
- Escalado automático de nodos EC2
- Optimización de costos con Spot Instances
- Balanceo entre zonas de disponibilidad

### 2.2 Balanceo de Carga

**Niveles de Balanceo:**

1. **Application Load Balancer (L7)**
   - Routing basado en path/host
   - SSL/TLS termination
   - Health checks avanzados

2. **Ingress Controller (L7)**
   - Routing interno en Kubernetes
   - Rate limiting
   - Autenticación en edge

3. **Service Mesh (L4/L7)**
   - Load balancing inteligente
   - Circuit breaker
   - Retry automático

### 2.3 Resiliencia y Alta Disponibilidad

**Estrategias Implementadas:**

**Multi-AZ Deployment:**
- Distribución en 3 zonas de disponibilidad
- Replicación automática de datos
- Failover automático

**Circuit Breaker Pattern:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: project-service
spec:
  host: project-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

**Backup y Recuperación:**
- RDS automated backups (35 días)
- Point-in-time recovery
- Cross-region replication para DR

**Health Checks:**
- Liveness probes: Detecta contenedores muertos
- Readiness probes: Controla tráfico a pods
- Startup probes: Maneja aplicaciones de inicio lento

### 2.4 Justificación de Kubernetes

**Ventajas Técnicas:**

**Orquestación Nativa:**
- Gestión automática del ciclo de vida
- Self-healing capabilities
- Declarative configuration

**Service Discovery:**
- DNS interno automático
- Service mesh integration
- Load balancing nativo

**Rolling Updates:**
- Zero-downtime deployments
- Rollback automático en caso de fallo
- Canary deployments

**Resource Management:**
- Scheduling inteligente
- Resource quotas
- QoS classes

**Ecosystem Maduro:**
- Integración con herramientas DevOps
- Helm para package management
- Operators para aplicaciones complejas

---