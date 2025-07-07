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

## 3. Contenedorización y Orquestación

### 3.1 Estrategia de Contenedorización

**Docker como Estándar:**

**Multi-stage Builds:**
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 8080
USER node
CMD ["node", "server.js"]
```

**Optimizaciones:**
- Imágenes base Alpine para menor tamaño
- Layer caching para builds rápidos
- Non-root user para seguridad
- Health checks integrados

**Security Scanning:**
- Análisis automático de vulnerabilidades
- Políticas de seguridad en CI/CD
- Imágenes firmadas digitalmente

### 3.2 Orquestación con Kubernetes

**Componentes Clave:**

**Deployments:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: project-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: project-service
  template:
    metadata:
      labels:
        app: project-service
    spec:
      containers:
      - name: project-service
        image: promanage/project-service:v1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: host
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Services:**
- ClusterIP para comunicación interna
- NodePort para acceso externo de desarrollo
- LoadBalancer para producción

**ConfigMaps y Secrets:**
- Gestión centralizada de configuración
- Separación de secrets sensibles
- Versionado de configuración

**Persistent Volumes:**
- Storage classes para diferentes tipos
- Backup automático
- Snapshot policies

### 3.3 Registro de Contenedores

**Amazon ECR como Solución:**

**Ventajas:**
- **Integración Nativa**: Seamless con EKS y otros servicios AWS
- **Security Scanning**: Análisis automático con AWS Inspector
- **Lifecycle Policies**: Gestión automática de imágenes antiguas
- **IAM Integration**: Control granular de acceso
- **Geo-replication**: Replicación cross-region automática

**Configuración:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ecr-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-config>
```

**CI/CD Integration:**
- Push automático en merge a main
- Tagging semántico
- Vulnerability scanning en pipeline

---

## 4. Diagrama de Arquitectura

### 4.1 Arquitectura de Alto Nivel

El diagrama Mermaid incluido muestra la arquitectura completa con:

**Capa de Presentación:**
- Web Application (React/Angular)
- Mobile Application (React Native/Flutter)
- API Clients (terceros)

**Capa de Gateway:**
- Application Load Balancer
- API Gateway Service
- Rate limiting y autenticación

**Capa de Servicios:**
- 5 microservicios independientes
- Service Mesh para comunicación
- Sidecar proxies para seguridad

**Capa de Datos:**
- Bases de datos especializadas por servicio
- Cache distribuido
- Almacenamiento de archivos

**Capa de Infraestructura:**
- Kubernetes cluster gestionado
- Servicios AWS gestionados
- Monitoreo y observabilidad

### 4.2 Flujos de Comunicación

**Flujo de Autenticación:**
1. Cliente → ALB → API Gateway
2. API Gateway → Auth Service
3. Auth Service → User Service (validación)
4. Respuesta con JWT token

**Flujo de Gestión de Proyectos:**
1. Cliente autenticado → API Gateway
2. Validación JWT en API Gateway
3. Routing a Project Service
4. Project Service → Database
5. Respuesta con datos del proyecto

**Flujo de Archivos:**
1. Cliente → File Service
2. File Service → S3 (almacenamiento)
3. Metadata → File Database
4. Respuesta con URL firmada

### 4.3 Herramientas Integradas

**Bases de Datos:**
- **PostgreSQL (RDS)**: Datos transaccionales
- **Redis (ElastiCache)**: Cache y sesiones
- **S3**: Almacenamiento de archivos

**Monitoreo:**
- **CloudWatch**: Métricas y logs AWS
- **Prometheus**: Métricas de Kubernetes
- **Grafana**: Dashboards y visualización
- **Jaeger**: Distributed tracing

**Seguridad:**
- **AWS IAM**: Control de acceso
- **AWS WAF**: Protección web
- **Service Mesh**: mTLS automático
- **Network Policies**: Segmentación de red

---

## 5. Consideraciones de Seguridad

### 5.1 Seguridad en Capas

**Capa de Red:**
- VPC con subnets privadas
- Security Groups restrictivos
- Network ACLs
- WAF en ALB

**Capa de Aplicación:**
- JWT con rotación automática
- Rate limiting por cliente
- Input validation
- HTTPS/TLS everywhere

**Capa de Datos:**
- Encryption at rest
- Encryption in transit
- Database credentials rotation
- Backup encryption

### 5.2 Compliance y Auditoria

**Logging:**
- Audit logs centralizados
- Retention policies
- Log analysis con CloudWatch Insights

**Compliance:**
- SOC 2 Type II ready
- GDPR compliance
- Data residency controls

---

## 6. Estimación de Costos

### 6.1 Costos Mensuales Estimados (USD)

**Compute:**
- EKS Cluster: $75
- EC2 Instances (t3.medium x 3): $95
- ALB: $25

**Storage:**
- RDS PostgreSQL (db.t3.micro): $25
- ElastiCache Redis: $15
- S3 Storage (100GB): $3

**Networking:**
- Data Transfer: $10
- NAT Gateway: $45

**Total Estimado: ~$293/mes**

### 6.2 Optimizaciones de Costo

- Spot Instances para desarrollo
- Reserved Instances para producción
- S3 Intelligent Tiering
- Auto-scaling para ajustar recursos

---