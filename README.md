Propuesta de Arquitectura Tecnológica para el Sistema Integral de Atención Ciudadana “Conecta360”

Contexto
El Sistema Integral de Atención Ciudadana “Conecta360” tiene como propósito centralizar, modernizar y hacer trazable la gestión de solicitudes, quejas, reclamos, sugerencias y servicios al ciudadano, habilitando una experiencia omnicanal y una operación eficiente para las áreas responsables. En ese contexto, el presente documento describe la Propuesta de Arquitectura Tecnológica que servirá como marco de referencia para el diseño, construcción, integración, despliegue y evolución de la solución.
La propuesta abarca los componentes de aplicación y datos, integración con sistemas internos y externos, seguridad y cumplimiento, gobierno de APIs, infraestructura y operación (incluyendo consideraciones de disponibilidad, rendimiento, monitoreo y continuidad). Está dirigida a actores clave como negocio, arquitectura empresarial/tecnológica, desarrollo, ciberseguridad, operaciones y proveedores, y busca alinear la implementación con buenas prácticas y estándares, asegurando una base escalable, resiliente y sostenible para el ciclo de vida de Conecta360.
Definición del esquema tecnológico
Componente	Tecnología seleccionada	Justificación	Alternativa evaluada
Backend	NestJS	Alta compatibilidad con microservicios, lo que facilita el mantenimiento a largo plazo y una arquitectura modular y escalable, ideal para aplicaciones grandes.	Spring Boot
Orquestación BPM	Camunda 8	Permite modelar flujos gubernamentales complejos que requieren aprobaciones humanas de forma visual.	Bizagi
Base de Datos Principal	PostgreSQL	Soporta JSONB para metadatos flexibles, extensiones GIS y licenciamiento open source. Además, ofrece robustez transaccional para asegurar que ningún ticket se pierda.	MySQL / Oracle
Base de datos no relacional	MongoDB	Adecuada para almacenar logs de incidencias y estructuras JSON variables provenientes de sistemas legados.	 
BI/Analítica	Apache Superset + BigQuery	Permite exportación hacia Power BI mediante conector, con un costo operativo significativamente menor.	Power BI Premium
Mensajería	Apache Kafka	Capacidad para procesar millones de eventos por segundo con persistencia.	RabbitMQ
Cache/Sesión	Redis	Alto rendimiento y baja latencia, con reducción de carga en la base de datos y gestión eficiente de sesiones.	Memcached
Frontend	React.js + Next.js	Optimización SEO para el portal web y carga progresiva.	Angular, Vue.js
Identidad (SSO)	Keycloak	Open source, compatible con OIDC y SAML para integrarse con el ID nacional.	Auth0
IA / Chatbot	Python (FastAPI + LangChain)	Agilidad para implementar modelos de lenguaje que clasifiquen tickets automáticamente.	 
API Gateway	Kong	Conectividad rápida para APIs, ideal para microservicios, con capacidades de escalabilidad, seguridad y gestión de tráfico.	AWS API Gateway
Observabilidad	Prometheus + Grafana + Loki	No tiene costo por volumen de métricas, cubre toda la infraestructura y ofrece observabilidad end-to-end con auditoría completa.	Datadog
CI/CD	GitLab	Automatización eficiente del desarrollo, seguridad integrada y detección temprana de errores.	Azure Pipelines
Patrón arquitectónico
Para la solución propuesta, se seleccionan los siguientes patrones arquitectónicos, orientados a garantizar escalabilidad, mantenibilidad, interoperabilidad y gobernabilidad tecnológica.
Layered Architecture, para mantener orden y gobernabilidad. 
Microservicios, se eligió una arquitectura de microservicios porque cada módulo (ciudadanos, casos, notificaciones, analítica) tiene ciclos de vida, equipos y escala independientes. La comunicación asíncrona vía Kafka desacopla los servicios y permite absorber picos de hasta 500,000 solicitudes diarias sin bloqueo.
Event-Driven Architecture, para garantizar la escalabilidad y la interoperabilidad demandada.
El patrón CQRS (Command Query Responsibility Segregation) se aplica en el Case Service: las escrituras van a PostgreSQL (fuente de verdad) y las lecturas analíticas a Elasticsearch o el Data Warehouse, lo que mantiene <1.5s de respuesta sin degradar la base transaccional.
Outbox Pattern: Para asegurar que un caso se guarde en la DB y se publique en Kafka de forma atómica, evitando pérdida de datos.
Adapter/Sidecar: Para sistemas legados sin API, se despliega un "adaptador" que consulta sus bases de datos y traduce la información al estándar de Conecta360.
API Gateway, puerta de acceso única, centraliza todas las solicitudes, evita acceso directo a servicios interno o legacy, representa control y gobierno.
Arquitectura tecnológica para implementar
Diagrama inicial de contenedores
 
Diagrama de componentes de alto nivel
 
 
Diagrama de contexto
 
 
Diagramas orientados al BPM
 
 
 
Alta disponibilidad geográfica
 
Diagrama de identidades
 
Arquitectura de base de datos

 

Diseño y estructura de APIs RESTful y asíncronas
REST: endpoints principales
Base URL: https://api.conecta360.gov.do/v1
Autenticación: Authorization: Bearer <JWT> en todos los endpoints.
POST /cases — Crear nuevo caso
json
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
GET /cases/{case_number} — Consultar estado
json
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
GET /dashboard/kpis — Dashboard global (rol supervisor/admin)
json
// Response 200 OK
{
  "period": "2025-07",
  "total_cases": 48320,
  "resolved": 41200,
  "resolution_rate": 0.853,
  "avg_resolution_hours": 22.4,
  "by_dependency": [
    { "code": "energia", "cases": 12400, "avg_hours": 18.2, "satisfaction": 4.1 },
    { "code": "transporte", "cases": 9800, "avg_hours": 31.5, "satisfaction": 3.7 }
  ],
  "citizen_satisfaction_avg": 3.9
}
AsyncAPI: eventos Kafka
Topic cases.created
yaml
# AsyncAPI 2.6 (simplificado)
channels:
  cases.created:
    subscribe:
      summary: Nuevo caso ciudadano registrado
      message:
        payload:
          type: object
          properties:
            case_id:      { type: string, format: uuid }
            case_number:  { type: string, example: "C360-2025-00041821" }
            citizen_id:   { type: string, format: uuid }
            category:     { type: string }
            dependency:   { type: string }
            created_at:   { type: string, format: date-time }
          required: [case_id, case_number, citizen_id, category, dependency, created_at]
Topic cases.status_changed
yaml
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




Linea de Tiempo

El objetivo de esta línea de tiempo es que se ilustre como se estaría abordando dicho proyecto de una manera ejemplificada no necesariamente acorde a la realidad.

 




Referencias

https://app.diagrams.net/

https://mermaid.live/

https://plantuml.com/

