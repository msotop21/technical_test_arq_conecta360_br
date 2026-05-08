# Propuesta de Arquitectura Tecnológica — Sistema Integral de Atención Ciudadana "Conecta360"

---

## 📋 Contexto

El Sistema Integral de Atención Ciudadana **"Conecta360"** tiene como propósito centralizar, modernizar y hacer trazable la gestión de solicitudes, quejas, reclamos, sugerencias y servicios al ciudadano, habilitando una experiencia omnicanal y una operación eficiente para las áreas responsables.

La propuesta abarca los componentes de aplicación y datos, integración con sistemas internos y externos, seguridad y cumplimiento, gobierno de APIs, infraestructura y operación (incluyendo consideraciones de disponibilidad, rendimiento, monitoreo y continuidad).

Está dirigida a actores clave como negocio, arquitectura empresarial/tecnológica, desarrollo, ciberseguridad, operaciones y proveedores, y busca alinear la implementación con buenas prácticas y estándares, asegurando una base escalable, resiliente y sostenible para el ciclo de vida de Conecta360.

---

## 🛠️ Esquema Tecnológico

| Componente | Tecnología Seleccionada | Justificación | Alternativa Evaluada |
|---|---|---|---|
| **Backend** | NestJS | Alta compatibilidad con microservicios, arquitectura modular y escalable, ideal para aplicaciones grandes. | Spring Boot |
| **Orquestación BPM** | Camunda 8 | Permite modelar flujos gubernamentales complejos con aprobaciones humanas de forma visual. | Bizagi |
| **Base de Datos Principal** | PostgreSQL | Soporta JSONB para metadatos flexibles, extensiones GIS y licenciamiento open source. Robustez transaccional. | MySQL / Oracle |
| **Base de Datos No Relacional** | MongoDB | Almacenamiento de logs de incidencias y estructuras JSON variables de sistemas legados. | — |
| **BI / Analítica** | Apache Superset + BigQuery | Exportación hacia Power BI mediante conector, con costo operativo significativamente menor. | Power BI Premium |
| **Mensajería** | Apache Kafka | Procesa millones de eventos por segundo con persistencia. | RabbitMQ |
| **Caché / Sesión** | Redis | Alto rendimiento, baja latencia y gestión eficiente de sesiones. | Memcached |
| **Frontend** | React.js + Next.js | Optimización SEO para el portal web y carga progresiva. | Angular, Vue.js |
| **Identidad (SSO)** | Keycloak | Open source, compatible con OIDC y SAML para integrarse con el ID nacional. | Auth0 |
| **IA / Chatbot** | Python (FastAPI + LangChain) | Implementación ágil de modelos de lenguaje para clasificar tickets automáticamente. | — |
| **API Gateway** | Kong | Conectividad rápida para APIs, con capacidades de escalabilidad, seguridad y gestión de tráfico. | AWS API Gateway |
| **Observabilidad** | Prometheus + Grafana + Loki | Sin costo por volumen de métricas, observabilidad end-to-end con auditoría completa. | Datadog |
| **CI/CD** | GitLab | Automatización eficiente, seguridad integrada y detección temprana de errores. | Azure Pipelines |

---

## 🏗️ Patrón Arquitectónico

Se seleccionan los siguientes patrones orientados a garantizar escalabilidad, mantenibilidad, interoperabilidad y gobernabilidad tecnológica:

### Layered Architecture
Mantiene orden y gobernabilidad en la solución.

### Microservicios
Cada módulo (ciudadanos, casos, notificaciones, analítica) tiene ciclos de vida, equipos y escala independientes. La comunicación asíncrona vía Kafka desacopla los servicios y permite absorber picos de hasta **500,000 solicitudes diarias** sin bloqueo.

### Event-Driven Architecture
Garantiza la escalabilidad e interoperabilidad demandada.

### CQRS (Command Query Responsibility Segregation)
Aplicado en el **Case Service**: las escrituras van a PostgreSQL (fuente de verdad) y las lecturas analíticas a Elasticsearch o el Data Warehouse, manteniendo **< 1.5s de respuesta** sin degradar la base transaccional.

### Outbox Pattern
Asegura que un caso se guarde en la DB y se publique en Kafka de forma atómica, evitando pérdida de datos.

### Adapter / Sidecar
Para sistemas legados sin API, se despliega un adaptador que consulta sus bases de datos y traduce la información al estándar de Conecta360.

### API Gateway
Puerta de acceso única que centraliza todas las solicitudes, evita acceso directo a servicios internos o legacy, y representa control y gobierno.

---

## 📐 Arquitectura Tecnológica

### Diagramas
- Diagrama inicial de contenedores
- Diagrama de componentes de alto nivel
- Diagrama de contexto
- Diagramas orientados al BPM
- Alta disponibilidad geográfica
- Diagrama de identidades

> 📌 Los diagramas pueden visualizarse en [app.diagrams.net](https://app.diagrams.net/) o [mermaid.live](https://mermaid.live/).

---

## 🗄️ Arquitectura de Base de Datos

> *(Ver diagrama de base de datos adjunto)*

---

## 🔌 Diseño y Estructura de APIs RESTful y Asíncronas

**Base URL:** `https://api.conecta360.gov.do/v1`

**Autenticación:** `Authorization: Bearer <JWT>` en todos los endpoints.

---

### REST: Endpoints Principales

#### `POST /cases` — Crear nuevo caso

```json
// Request
{
  "citizen_id": "c7f3a1b2-...",
  "category": "alumbrado_publico",
  "subcategory": "lampara_apagada",
  "description": "Farola en Av. Central km 3 sin luz desde hace 4 días",
  "location": { "lat": 9.9281, "lng": -84.0907 },
  "channel": "web",
  "attachments": ["https://cdn.c360.gv/uploads/img_001.jpg"]
}

// Response 201 Created
{
  "case_id": "a1b2c3d4-...",
  "case_number": "C360-2025-00041821",
  "status": "received",
  "dependency": "energia_electrica",
  "sla_deadline": "2025-07-14T18:00:00Z",
  "tracking_url": "https://conecta360.gv/track/C360-2025-00041821"
}
```

---

#### `GET /cases/{case_number}` — Consultar estado

```json
// Response 200 OK
{
  "case_number": "C360-2025-00041821",
  "status": "in_progress",
  "category": "alumbrado_publico",
  "assigned_to": { "name": "Lic. María López", "dependency": "Energía Eléctrica" },
  "events": [
    { "type": "received",    "at": "2025-07-10T10:32:00Z", "note": "Caso registrado" },
    { "type": "assigned",    "at": "2025-07-10T10:34:00Z", "note": "Derivado a cuadrilla 3" },
    { "type": "in_progress", "at": "2025-07-11T08:00:00Z", "note": "Técnico en camino" }
  ],
  "sla_deadline": "2025-07-14T18:00:00Z",
  "sla_status": "on_track"
}
```

---

#### `GET /dashboard/kpis` — Dashboard global *(rol: supervisor/admin)*

```json
// Response 200 OK
{
  "period": "2025-07",
  "total_cases": 48320,
  "resolved": 41200,
  "resolution_rate": 0.853,
  "avg_resolution_hours": 22.4,
  "by_dependency": [
    { "code": "energia",    "cases": 12400, "avg_hours": 18.2, "satisfaction": 4.1 },
    { "code": "transporte", "cases": 9800,  "avg_hours": 31.5, "satisfaction": 3.7 }
  ],
  "citizen_satisfaction_avg": 3.9
}
```

---

### AsyncAPI: Eventos Kafka

#### Topic `cases.created`

```yaml
# AsyncAPI 2.6 (simplificado)
channels:
  cases.created:
    subscribe:
      summary: Nuevo caso ciudadano registrado
      message:
        payload:
          type: object
          properties:
            case_id:     { type: string, format: uuid }
            case_number: { type: string, example: "C360-2025-00041821" }
            citizen_id:  { type: string, format: uuid }
            category:    { type: string }
            dependency:  { type: string }
            created_at:  { type: string, format: date-time }
          required: [case_id, case_number, citizen_id, category, dependency, created_at]
```

#### Topic `cases.status_changed`

```yaml
channels:
  cases.status_changed:
    subscribe:
      summary: Cambio de estado en un caso existente
      message:
        payload:
          type: object
          properties:
            case_id:    { type: string, format: uuid }
            old_status: { type: string, enum: [received, assigned, in_progress, resolved, closed] }
            new_status: { type: string, enum: [received, assigned, in_progress, resolved, closed] }
            actor_id:   { type: string, format: uuid }
            note:       { type: string }
            occurred_at: { type: string, format: date-time }
```

---

## 📅 Línea de Tiempo

> El objetivo de esta línea de tiempo es ilustrar cómo se abordaría el proyecto de manera ejemplificada, no necesariamente acorde a la realidad.

| Fase | Descripción | Estado |
|---|---|---|
| Fase 1 | Levantamiento de requerimientos y arquitectura base | 🔲 Planificado |
| Fase 2 | Desarrollo de microservicios core (casos, ciudadanos) | 🔲 Planificado |
| Fase 3 | Integración BPM, mensajería y notificaciones | 🔲 Planificado |
| Fase 4 | Módulo de analítica, BI y dashboard | 🔲 Planificado |
| Fase 5 | Pruebas, seguridad y despliegue productivo | 🔲 Planificado |

---

## 📚 Referencias

- [app.diagrams.net](https://app.diagrams.net/)
- [mermaid.live](https://mermaid.live/)
- [plantuml.com](https://plantuml.com/)
